# 创建任务

深入探讨我们在第2章，行为树和黑板中看过的概念，任务是我们的AI代理可以执行的单个动作。一些例子包括走到特定位置、执行/运行EQS、定位某物、追逐玩家等。所有这些动作可能会失败或成功。任务的最终结果随后被带回行为树，根据选择器和序列的规则。&#x20;

任务不一定必须在帧中执行，但它可以无限期地扩展。实际上，直到任务报告失败或成功，任务才算完成。 然而，它们可以被外部节点（如装饰器）中断/中止。&#x20;

当您创建任务时，无论是使用蓝图还是C++，都需要覆盖一些函数。由于蓝图更容易，并且与我们使用C++的概念相同，我们将首先看看系统如何在蓝图中工作。

**创建蓝图任务**

要创建蓝图任务，我们有几个选项可用。最简单的方法是在行为树编辑器中，我们按下顶部栏中的“新任务”按钮，如下图所示：&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

但是，您需要手动重命名文件并将其放置到您希望它所在的文件夹中。&#x20;

创建任务的另一种方法是创建一个从BTTask\_BlueprintBase继承的新蓝图，如下图所示：

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

按照惯例，任务前缀为 "BBTask\_"（代表行为树任务）。例如，我们可以将任务命名为 BTTask\_BPMyFirstTask：

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

创建蓝图任务后，我们可以覆盖三种类型的函数：

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* Receive Execute：任务开始时调用此函数，您可以在此实现任务的所有初始化。
* Receive Tick：每次任务 tick 时调用此函数，因此您可以使用它不断执行某些操作。然而，由于可能有许多代理执行许多行为树，建议尽量保持此 Tick 函数简短，或者为了性能原因根本不实现它，而是使用计时器或委托来处理任务。
* Receive Abort：每次任务正在执行但行为树请求中止时调用此函数。您需要使用此函数清理任务（例如，恢复某些黑板值）。

在Blueprint中，这三个功能以两种形式存在，AI和非AI，也被称为通用（例如Receive Execute和Receive Execute AI）。它们之间没有太大差别。如果只实现了一个（建议实现AI版本以保持项目一致性），那么就会调用该功能。否则，将调用最方便的那个，这意味着当Pawn被AI控制器占据时，调用AI版本，在其他所有情况下调用非AI版本。当然，您的大部分情况可能是行为树运行在AI控制器之上，因此非AI版本适用于非常特定和罕见的情况。

到目前为止，系统无法理解任务何时完成执行或完成中止后的清理工作。因此，您需要调用两个函数：

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

* Finish Execute：这将表示任务已经完成执行。它有一个布尔参数，表示任务是成功（true值）还是失败（false值）。
* Finish Abort：这将表示任务已经完成中止。它没有参数。

请注意，如果您不调用这两个函数，任务将永远悬在那里，这不是期望的行为。尽管建议在Receive Abort事件结束时调用Finish Abort函数，但在某些情况下，您可能需要多于一帧的时间来清理。在这种情况下，您可以在其他地方调用Finish Abort（例如在委托中）。

{% hint style="info" %}
还有其他完成执行任务的方法，例如使用AI消息，但我们不在本书中讨论。&#x20;
{% endhint %}

这就是创建任务所需了解的全部内容。你只需创建所需的图表，并在完成时调用执行完成节点（成功或失败）。 我们将在接下来的三个章节中查看创建新任务的具体示例。&#x20;

**在C++中创建任务**

在C++中创建任务的概念与其蓝图对应物相同。&#x20;

首先，要创建新的C++任务，我们需要创建一个从BTTaskNode继承的C++类，如下图所示：

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

就像蓝图任务一样，约定是使用"BTTask\_"（行为树任务）作为任务的前缀。因此，我们可以将任务命名为"BTTask\_MyFirstTask"：&#x20;

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

创建任务后，需要重写一些功能与蓝图中非常相似的函数。然而，它们之间也有一些差异。&#x20;

主要区别之一是如何报告任务已完成执行（或已中止）。在这些情况下，有一个特殊的枚举结构叫做EBTNodeResult。 它需要通过一个函数返回，以便行为树"知道"是否需要继续调用任务。这个结构可以有四个值：&#x20;

* 成功：任务以成功结束
* 失败：任务以失败结束
* 中止：任务已中止
* 进行中：任务尚未完成

另一个区别在于，蓝图中的Receive Execute的孪生函数必须结束，因此它需要返回一个EBTNodeResult结构来沟通并说明任务是否已完成，或者是否需要多于一帧的时间。如果是这样，那么将调用其他函数，我们将会看到。

此外，在C++中，您可以使用一些在蓝图中无法使用的特殊概念和结构。例如，您可以访问NodeMemory，它为已执行的任务持有特定的内存。要正确使用此结构，请查看引擎源代码，特别是本节末尾建议的文件。

最后一个区别是，没有AI和非AI（通用）版本的函数。您必须自己判断是否有AI控制器以及如何处理（如果需要处理）。

函数如下（直接从引擎源代码中获取，两个最重要的函数加粗显示）：

<pre class="language-cpp" data-line-numbers><code class="lang-cpp">/** starts this task, should return Succeeded, Failed or InProgress
   * (use FinishLatentTask() when returning InProgress)
   * this function should be considered as const (don't modify state of
<strong>  object) if node is not instanced! */
</strong>  virtual EBTNodeResult::Type ExecuteTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory);
protected:
  /** aborts this task, should return Aborted or InProgress
   * (use FinishLatentAbort() when returning InProgress)
   * this function should be considered as const (don't modify state of
  object) if node is not instanced! */
  virtual EBTNodeResult::Type AbortTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory);
public:
#if WITH_EDITOR
  virtual FName GetNodeIconName() const override;
#endif // WITH_EDITOR
  virtual void OnGameplayTaskDeactivated(UGameplayTask&#x26; Task) override;
  /** message observer's hook */
  void ReceivedMessage(UBrainComponent* BrainComp, const FAIMessage&#x26; Message);
  /** wrapper for node instancing: ExecuteTask */
  EBTNodeResult::Type WrappedExecuteTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory) const;
  /** wrapper for node instancing: AbortTask */
  EBTNodeResult::Type WrappedAbortTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory) const;
  /** wrapper for node instancing: TickTask */
  void WrappedTickTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8*
  NodeMemory, float DeltaSeconds) const;
  /** wrapper for node instancing: OnTaskFinished */
  void WrappedOnTaskFinished(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory, EBTNodeResult::Type TaskResult) const;
  /** helper function: finish latent executing */
  void FinishLatentTask(UBehaviorTreeComponent&#x26; OwnerComp, EBTNodeResult::Type TaskResult) const;
  /** helper function: finishes latent aborting */
  void FinishLatentAbort(UBehaviorTreeComponent&#x26; OwnerComp) const;
  /** @return true if task search should be discarded when this task is selected to execute but is already running */
  bool ShouldIgnoreRestartSelf() const;
  /** service nodes */
  UPROPERTY()
  TArray&#x3C;UBTService*> Services;
protected:
  /** if set, task search will be discarded when this task is selected to execute but is already running */
  UPROPERTY(EditAnywhere, Category=Task)
  uint32 bIgnoreRestartSelf : 1;
  /** if set, TickTask will be called */
  uint32 bNotifyTick : 1;
  /** if set, OnTaskFinished will be called */
  uint32 bNotifyTaskFinished : 1;
  /** ticks this task
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void TickTask(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory, float DeltaSeconds);
  /** message handler, default implementation will finish latent execution/abortion
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnMessage(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory, FName Message, int32 RequestID, bool bSuccess);
  /** called when task execution is finished
   * this function should be considered as const (don't modify state of object) if node is not instanced! */
  virtual void OnTaskFinished(UBehaviorTreeComponent&#x26; OwnerComp, uint8* NodeMemory, EBTNodeResult::Type TaskResult);
  /** register message observer */
  void WaitForMessage(UBehaviorTreeComponent&#x26; OwnerComp, FName MessageType)
const;
  void WaitForMessage(UBehaviorTreeComponent&#x26; OwnerComp, FName MessageType, int32 RequestID) const;
  /** unregister message observers */
  void StopWaitingForMessages(UBehaviorTreeComponent&#x26; OwnerComp) const;
</code></pre>

如你所见，这里有很多代码，一开始可能会有点混乱。然而，如果你已经很好地理解了蓝图，那么跳到理解C++函数应该会容易得多。例如，ExecuteTask()函数开始执行任务，但如果它返回任务仍在进行中，则不会完成任务。

以下是引擎源代码中的一条注释，可能有助于澄清这一点：

{% code lineNumbers="true" %}
```cpp
/**
 * Task are leaf nodes of behavior tree, which perform actual actions
 *
 * Because some of them can be instanced for specific AI, following virtual functions are not marked as const:
 * - ExecuteTask
 * - AbortTask
 * - TickTask
 * - OnMessage
 *
 * If your node is not being instanced (default behavior), DO NOT change any properties of object within those functions!
 * Template nodes are shared across all behavior tree components using the same tree asset and must store
 * their runtime properties in provided NodeMemory block (allocation size determined by GetInstanceMemorySize() )
 *
 */
```
{% endcode %}

我所知道的两种更好地了解如何创建C++任务的方法是：自己创建一个，或者阅读其他任务的源代码。例如，你可以阅读引擎源代码中的BTTask\_MoveTo.cpp文件，以获得关于如何创建C++任务的完整示例。不要气馁，因为使用C++很棒！

无论如何，我们将在接下来的三章中从头开始介绍创建C++任务的过程。
