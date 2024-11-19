# 创建装饰器

回顾第2章《行为树与黑板》，装饰器是一个条件节点（也可以看作是一个门），它控制着其所附加的子分支的执行流程（如果执行本来会进入子分支的话）。&#x20;

与扩展/创建任务的方式类似，我们可以扩展/创建一个装饰器。 首先，我们将深入探讨如何在蓝图中进行操作，然后再讨论如何在C++中进行扩展。&#x20;

**在蓝图中创建装饰器**

要创建蓝图装饰器，就像我们为任务所做的那样，您可以在行为树编辑器的顶部栏中按“新建装饰器”按钮，如下图所示：

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

或者，你可以生成一个继承自BTDecorator\_BlueprintBase的蓝图类，如下图所示：&#x20;

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

无论如何，命名规范是在装饰器前加上"BTDecorator\_"（代表行为树装饰器）。例如，我们可以将我们的类命名为BTDecorator\_BPMyFirstDecorator：

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

关于任务，所有可重写的函数都有两种形式：AI和非AI。概念完全相同。如果只实现了其中一个（为了保持项目的一致性，建议重写AI版本），那么就会调用该函数。如果两者都实现了，当Pawn被AI控制器占据时，调用AI函数，在所有其他情况下则调用非AI函数。

以下是Decorator可以扩展的六个函数：

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

* 执行条件检查：这是最重要的功能，也是唯一一个你可能需要重写的功能（如果你没有需要处理的动态事物）。它返回一个布尔值，表示条件检查是否成功。
* 接收执行开始：当底层节点（复合节点或任务节点）的执行开始时调用此功能。使用此功能初始化装饰器。
* 接收执行结束：当底层节点（复合节点或任务节点）的执行完成时调用此功能。使用此功能清理装饰器。
* 接收滴答：这是滴答函数，以防你需要不断更新某些内容。从性能角度来看，不建议将其用于繁重操作，最好不要使用它（例如，使用计时器或委托）。
* 接收观察者激活：顾名思义，当观察者被激活时调用此功能。
* 接收观察者停用：顾名思义，当观察者被停用时调用此功能。

如你所见，装饰器非常简单（至少在蓝图中是这样）；主要是，你只需要重写/实现执行条件检查功能，该功能返回一个布尔值：&#x20;

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

无论如何，我们将在接下来的三个章节中查看从零开始创建蓝图装饰器的具体示例。

**在C++中创建装饰器**

与我们在C++中扩展任务的方式非常相似，你也可以在C++中扩展装饰器。要继承的基类是BTDecorator，如下图所示：

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

通常，约定是以"BTDecorator\_"（行为树装饰器）为装饰器前缀。我们的装饰器可能的一个名字是"BTDecorator\_MyFirstDecorator"：

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

直接进入C++，这些是可重写的函数，摘自引擎源代码（有很多）：

{% code lineNumbers="true" %}
```cpp
 /** wrapper for node instancing: CalculateRawConditionValue */
  bool WrappedCanExecute(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const;
  /** wrapper for node instancing: OnNodeActivation */
  void WrappedOnNodeActivation(FBehaviorTreeSearchData& SearchData) const;
  /** wrapper for node instancing: OnNodeDeactivation */
  void WrappedOnNodeDeactivation(FBehaviorTreeSearchData& SearchData, EBTNodeResult::Type NodeResult) const;
  /** wrapper for node instancing: OnNodeProcessed */
  void WrappedOnNodeProcessed(FBehaviorTreeSearchData& SearchData, EBTNodeResult::Type& NodeResult) const;
  /** @return flow controller's abort mode */
  EBTFlowAbortMode::Type GetFlowAbortMode() const;
  /** @return true if condition should be inversed */
  bool IsInversed() const;
  virtual FString GetStaticDescription() const override;
  /** modify current flow abort mode, so it can be used with parent composite */
void UpdateFlowAbortMode();
  /** @return true if current abort mode can be used with parent composite */
  bool IsFlowAbortModeValid() const;
protected:
  /** if set, FlowAbortMode can be set to None */
  uint32 bAllowAbortNone : 1;
  /** if set, FlowAbortMode can be set to LowerPriority and Both */
  uint32 bAllowAbortLowerPri : 1;
  /** if set, FlowAbortMode can be set to Self and Both */
  uint32 bAllowAbortChildNodes : 1;
  /** if set, OnNodeActivation will be used */
  uint32 bNotifyActivation : 1;
  /** if set, OnNodeDeactivation will be used */
  uint32 bNotifyDeactivation : 1;
  /** if set, OnNodeProcessed will be used */
  uint32 bNotifyProcessed : 1;
  /** if set, static description will include default description of inversed condition */
  uint32 bShowInverseConditionDesc : 1;
private:
  /** if set, condition check result will be inversed */
  UPROPERTY(Category = Condition, EditAnywhere)
  uint32 bInverseCondition : 1;
protected:
  /** flow controller settings */
  UPROPERTY(Category=FlowControl, EditAnywhere)
  TEnumAsByte<EBTFlowAbortMode::Type> FlowAbortMode;
  void SetIsInversed(bool bShouldBeInversed);
  /** called when underlying node is activated
    * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnNodeActivation(FBehaviorTreeSearchData& SearchData);
  /** called when underlying node has finished
  * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnNodeDeactivation(FBehaviorTreeSearchData& SearchData, EBTNodeResult::Type NodeResult);
  /** called when underlying node was processed (deactivated or failed to activate)
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnNodeProcessed(FBehaviorTreeSearchData& SearchData, EBTNodeResult::Type& NodeResult);
  /** calculates raw, core value of decorator's condition. Should not include calling IsInversed */
  virtual bool CalculateRawConditionValue(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory) const;
  /** more "flow aware" version of calling RequestExecution(this) on owning behavior tree component
   * should be used in external events that may change result of CalculateRawConditionValue */
  void ConditionalFlowAbort(UBehaviorTreeComponent& OwnerComp, EBTDecoratorAbortRequest RequestMode) const;
```
{% endcode %}

此外，在引擎源代码中，我们可以找到以下注释，它解释了一些实现选择：

{% code lineNumbers="true" %}
```cpp
/**
 * Decorators are supporting nodes placed on parent-child connection, that receive notification about execution flow and can be ticked
 *
 * Because some of them can be instanced for specific AI, following virtual functions are not marked as const:
 *  - OnNodeActivation
 *  - OnNodeDeactivation
 *  - OnNodeProcessed
 *  - OnBecomeRelevant (from UBTAuxiliaryNode)
 *  - OnCeaseRelevant (from UBTAuxiliaryNode)
 *  - TickNode (from UBTAuxiliaryNode)
 *
 * If your node is not being instanced (default behavior), DO NOT change any properties of object within those functions!
 * Template nodes are shared across all behavior tree components using the same tree asset and must store
 * their runtime properties in provided NodeMemory block (allocation size determined by GetInstanceMemorySize() )
 *
 */
```
{% endcode %}

不幸的是，我们没有时间详细讲解所有的内容，但它们中的大部分都非常直观，所以你应该不难理解它们的含义。无论如何，我们将在接下来的三章中通过一个具体的例子来了解如何从头开始创建一个C++装饰器（我们将使用其中的许多函数）。
