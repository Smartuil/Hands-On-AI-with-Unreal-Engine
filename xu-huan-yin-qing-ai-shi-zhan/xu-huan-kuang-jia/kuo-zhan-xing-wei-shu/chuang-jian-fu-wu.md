# 创建服务

回顾一下第2章《行为树与黑板》，服务是一个节点，如果附加到子分支的父节点之一，它会不断运行。这个节点的主要用途是更新行为树黑板中的数据，它是你需要创建的节点之一，因为它们非常特定于你的游戏玩法。

与扩展任务和装饰器的方式类似，我们也可以扩展/创建服务。我们将首先了解如何在蓝图中实现扩展，然后也了解如何在C++中实现。

**在蓝图中创建服务**

就像你在任务和装饰器中所做的那样，你可以通过按照行为树编辑器顶部栏上的**新服务**按钮来创建一个新的**蓝图服务**，如下图所示：

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

或者，您可以生成继承自**BTService\_BlueprintBase**的蓝图类，如下图所示：

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

无论如何，命名规范是以"**BTService\_**"（代表行为树服务）为前缀。例如，我们可以将我们的类命名为类似BTService\_BPMyFirstService的东西：

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

就像任务和装饰器一样，所有可重写的函数都有两个不同的版本： AI 和非 AI（通用）。概念完全相同：如果只实现了其中一个 （为保持项目一致性，建议重写 AI 版本），那么就会调用该函数。如果两者都实现了，当 Pawn 被 AI 控制器占据时，调用 AI 函数，否则调用通用函数。&#x20;

这里有四个可重写的函数：

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

* 接收激活：当服务变为活动状态时调用。用于初始化服务。
* 接收滴答：当服务滴答时调用。主要是，服务连续执行某些操作（例如更新黑板变量），因此这是服务最重要的功能。从性能上讲，建议将其保持尽可能短。此外，回到行为树，可以调整服务滴答的频率（在最小值和最大值之间的随机值）。然而，理论上实现不应知道服务滴答的频率；它只需要提供一个“服务”。然后，服务的使用者，行为树，将决定它希望该服务的频率。
* 接收搜索开始：这是一个特殊情况，服务处于活动状态（所以你应该已经初始化了服务，但理论上，服务还不应该执行任何操作）。在搜索任务/节点之前调用此功能。实际上，行为树需要评估接下来要执行哪个任务或节点。在这样做时，行为树会检查可能的任务或节点上装饰器的条件。因此，在此功能中，您可以在搜索下一个任务或节点之前调整值，从而选择影响选择并使其成为正确选择的内容。
* 接收停用：当服务变为非活动状态时调用。用于清理服务。

主要，您需要在接收滴答函数中实现您的逻辑，以便可以不断更新黑板中的信息。服务是行为树和游戏世界之间的一层。 在接下来的三个章节中，我们将实现一个蓝图服务，在此过程中，我们将面临一个更实际、更具体的例子。

**在C++中创建服务**

与我们在C++中扩展任务和装饰器的方式非常相似，你也可以在C++中扩展服务。要继承的基类是BTService，如下图所示：

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

惯例是以"BTService\_"（行为树服务）为服务类名前缀。 我们类的一个可能名称是BTService\_MyFirstService：&#x20;

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

一旦我们在C++中创建了服务，其余部分与在C++中扩展/创建装饰器的方式非常相似。这里是要重写的函数（取自引擎源代码）：

{% code lineNumbers="true" %}
```cpp
  virtual FString GetStaticDescription() const override;
  void NotifyParentActivation(FBehaviorTreeSearchData& SearchData);
protected:
  // Gets the description of our tick interval
  FString GetStaticTickIntervalDescription() const;
  // Gets the description for our service
  virtual FString GetStaticServiceDescription() const;
  /** defines time span between subsequent ticks of the service */
  UPROPERTY(Category=Service, EditAnywhere, meta=(ClampMin="0.001"))
  float Interval;
  /** adds random range to service's Interval */
  UPROPERTY(Category=Service, EditAnywhere, meta=(ClampMin="0.0"))
  float RandomDeviation;
  /** call Tick event when task search enters this node (SearchStart will be called as well) */
  UPROPERTY(Category = Service, EditAnywhere, AdvancedDisplay)
  uint32 bCallTickOnSearchStart : 1;
  /** if set, next tick time will be always reset to service's interval when node is activated */
  UPROPERTY(Category = Service, EditAnywhere, AdvancedDisplay)
  uint32 bRestartTimerOnEachActivation : 1;
  /** if set, service will be notified about search entering underlying branch */
  uint32 bNotifyOnSearch : 1;
  /** update next tick interval
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void TickNode(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override;
  /** called when search enters underlying branch
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnSearchStart(FBehaviorTreeSearchData& SearchData);
#if WITH_EDITOR
  virtual FName GetNodeIconName() const override;
#endif // WITH_EDITOR
  /** set next tick time */
  void ScheduleNextTick(uint8* NodeMemory);
```
{% endcode %}

这里有一段开头注释（总是取自引擎源代码），解释了一些实现选择：

{% code lineNumbers="true" %}
```cpp
/**
 * Behavior Tree service nodes is designed to perform "background" tasks that update AI's knowledge.
 *
 * Services are being executed when underlying branch of behavior tree becomes active,
 * but unlike tasks they don't return any results and can't directly affect execution flow.
 *
 * Usually they perform periodical checks (see TickNode) and often store results in blackboard.
 * If any decorator node below requires results of check beforehand, use OnSearchStart function.
 * Keep in mind that any checks performed there have to be instantaneous!
 *
  * Other typical use case is creating a marker when specific branch is being executed
 * (see OnBecomeRelevant, OnCeaseRelevant), by setting a flag in blackboard.
 *
 * Because some of them can be instanced for specific AI, following virtual functions are not marked as const:
 * - OnBecomeRelevant (from UBTAuxiliaryNode)
 * - OnCeaseRelevant (from UBTAuxiliaryNode)
 * - TickNode (from UBTAuxiliaryNode)
 * - OnSearchStart
 *
 * If your node is not being instanced (default behavior), DO NOT change any properties of object within those functions!
 * Template nodes are shared across all behavior tree components using the same tree asset and must store
 * their runtime properties in provided NodeMemory block (allocation size determined by GetInstanceMemorySize() )
 */
```
{% endcode %}

不幸的是，我们无法详细介绍每个功能，但它们都非常容易理解。无论如何，我们将在接下来的三个章节中进一步探讨服务，当我们从头开始构建行为树时，以及当我们实现一个C++服务时。&#x20;
