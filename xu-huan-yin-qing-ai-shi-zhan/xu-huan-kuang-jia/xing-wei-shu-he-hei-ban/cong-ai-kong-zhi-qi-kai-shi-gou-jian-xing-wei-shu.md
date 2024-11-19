# 从AI控制器开始构建行为树

现在我们已经了解了行为树的基本概念和组成，让我们创建自己的行为树。回忆上一章，负责拥有Pawn并控制它的类是AI控制器。因此，我们的**行为树**应该在AI控制器上运行。

我们有两种方法可以做到这一点。第一种是使用蓝图。通常，即使你是一个程序员，最好使用蓝图创建**行为树**，因为逻辑非常简单，控制器也很简单。另一方面，如果你是一个C++爱好者，并且尽可能多地使用它，即使是小的任务，也不要担心——我将再次创建我们在蓝图中要做的相同的逻辑，但这次是用C++。无论如何，**行为树**资源应该在编辑器内创建和修改。你最终要编程的是与默认提供的节点不同的节点（我们会在本书后面看到这一点），但树本身总是在编辑器中制作的。

**创建行为树和黑板**

首先，我们需要创建四个蓝图类：**AI控制器、角色、行为树**和**黑板**。我们稍后会介绍AI控制器。如果你选择了两个第三人称模板之一，你应该已经有一个准备好的角色。因此，你只需要创建一个**行为树**和一个**黑板**。

在内容浏览器中，创建一个新文件夹并将其命名为Chapter2。这将有助于保持事情的整洁。然后，创建一个子文件夹并将其命名为AI。这样，我们可以保持项目的整洁，确保我们不会将本章的项目与其他非AI相关的类和/或对象混在一起。我们将把所有为AI创建的资产放在这个文件夹中。

**创建黑板**

现在，我们需要添加一个黑板，它应该一直在AI文件夹中。为此，转到**内容浏览器**并选择**添加新 > 人工智能 > 黑板**。

目前，我们将我们的黑板命名为`BB_MyFirstBlackboard`。在这里，我使用了命名约定来将所有黑板前缀为`BB_`。除非你有特定的理由不遵循这个命名约定，否则请使用它。这样做，你将与本书的其余部分保持同步。

{% hint style="info" %}
由于在同一行为树中不能有多个黑板，你可以在**黑板详情面板**中使用父子继承，如右侧截图所示：
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

**创建行为树**

让我们通过转到**内容浏览器**并选择**添加新人工智能 > 行为树**来添加一个行为树，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

目前，我们将我们的行为树命名为`BT_MyFirstBehaviorTree`。同样，在这里，我使用了一个特定的命名规范，给所有行为树资产加上了`BT_`前缀。再次强调，请遵循这个命名规范，除非你有特别的理由不这么做。

当你打开行为树窗口时，你会看到一个名为**Root**的节点，如下所示：

<figure><img src="../../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

根节点是行为树执行开始的地方（从上到下，从左到右）。**根**节点本身只有一个引用，那就是黑板，所以它不能连接到其他任何东西。它是树的顶端，所有后续节点都在它下面。

如果你从**根**节点拖动，你将能够添加复合节点：

<figure><img src="../../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

为此，行为树编辑器非常直观。你可以继续从节点中拖动以添加**复合**节点或**任务**节点。要添加**装饰器**或**服务**，你可以右键单击节点并选择“**添加装饰器**...”或“**添加服务**...”，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

最后，如果你点击一个节点，可以在详细信息面板中选择其参数（以下截图显示了移动到节点的示例）：

<figure><img src="../../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

**AI 控制器运行行为树**

下一步是从 AI 控制器运行行为树。通常，这是一个简单的任务，它在蓝图中实现（在其中可以直接引用特定的行为树）。即使我们有一个复杂的 C++ AI 控制器，我们也可以在蓝图中扩展该控制器并从蓝图中运行行为树。无论如何，如果硬引用不起作用（例如，您使用 C++ 或因为您想要更多的灵活性），那么您可以将行为树存储在需要运行该特定行为树的角色/Pawn 中，并在 AI 控制器拥有 Pawn 时检索它。

让我们探讨一下如何在蓝图中（我们将在变量中引用行为树，在其中我们可以决定默认值）和在 C++ 中（我们将在角色中存储行为树）实现这一点。

**蓝图中的 AI 控制器**

我们可以通过点击**添加新 | 蓝图类 | AI 控制器**来创建一个蓝图 AI 控制器。您将需要点击**所有类**并搜索 **AI 控制器**以访问它。您可以在以下屏幕截图中看到这方面的示例：

<figure><img src="../../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

目前，我们将我们的**AI 控制器**命名为`BP_MyFirstAIController`。双击它以打开蓝图编辑器。

首先，我们需要创建一个变量，以便我们可以存储我们的**行为树**。尽管保留行为树的引用并不是必须的，但这样做是一个很好的实践。要创建一个**变量**，我们需要在“我的蓝图”面板中按下“+”**变量**按钮，该按钮位于“**变量**”标签旁边，如下面的截图所示（请记住，您的光标需要放在“变量”标签上，按钮才会显示）：

<figure><img src="../../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure>

然后，作为变量类型，您需要选择行为树并给它一个名字，例如 **BehaviorTreeReference**。这就是您的变量应该的样子：

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

然后，在细节面板中，我们将设置默认值（请记住，要设置默认值，需要编译蓝图）：

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

然后，我们需要重写“**On Possess**”函数，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

最后，在AI控制器的事件“**On Possess**”中，我们需要开始运行/执行行为树。我们可以通过使用以下简单的节点来实现这一点，名为“**Run Behavior Tree**”：

<figure><img src="../../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

结果，您的AI控制器将能够执行存储在**BehaviorTreeReference**中的行为树。

**C++中的AI控制器**

如果您决定使用C++创建这个简单的AI控制器，让我们开始吧。我假设您的Unreal编辑器已经设置为使用C++（例如，您已经安装了Visual Studio，调试符号等）。这是一个参考链接，以便您可以开始：https://docs.unrealengine.com/en-us/Programming/QuickStart，并且您对C++在Unreal中的工作原理有基本的了解。这是一个命名约定的链接，以便您理解为什么代码中的一些类以字母为前缀：https://docs.unrealengine.com/en-us/Programming/Development/CodingStandard。

{% hint style="info" %}
在开始之前，请记住，为了在C++中处理AI，您需要在您的.cs文件（在这种情况下是UnrealAIBook.cs）中添加公共依赖项，并将GameplayTasks和AIModule添加为公共依赖项，如下面的代码所示：

```cpp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine", "InputCore",
    "HeadMountedDisplay", "GameplayTasks", "AIModule" });
```
{% endhint %}

这将确保您的代码将顺利编译而不会出现问题。

让我们创建一个新的C++类，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (20) (1) (1).png" alt="" width="293"><figcaption></figcaption></figure>

类需要继承自 **AIController** 类。你可能需要勾选右上角的“显示所有类”复选框，然后使用搜索栏，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

点击“下一步”，将类命名为 **MyFirstAIController**。此外，我建议你保持项目整洁。因此，点击“**选择文件夹**”按钮。Unreal 将提示你进入系统文件夹浏览器。在这里，创建一个名为 Chapter2 的文件夹，并在其中创建一个名为 AI 的子文件夹。选择这个文件夹作为你要存储我们即将创建的代码的地方。这就是你在点击“创建”之前的对话框样子：

<figure><img src="../../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

现在，点击“创建”并等待编辑器加载。你可能会看到类似这样的内容：

<figure><img src="../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

我们的代码结构与蓝图版本略有不同。实际上，我们不能直接从AI控制器类分配**行为树**（主要因为它很难直接引用）；相反，我们需要从角色中获取它。正如我之前提到的，当你使用蓝图时，这也是一个好方法，但由于我们选择了C++项目，我们应该查看一些代码。在Visual Studio中，打开UnrealAIBookCharacter.h文件，并在公共变量下方添加以下代码行：

```cpp
//** Behavior Tree for an AI Controller (Added in Chapter 2)
 UPROPERTY(EditAnywhere, BlueprintReadWrite, category=AI)
 UBehaviorTree* BehaviorTree;
```

对于仍不熟悉的人，这里有一大部分代码，以便你可以了解将前述代码放在类中的位置：

<pre class="language-cpp"><code class="lang-cpp">public： 
  
  AUnrealAIBookCharacter();
<strong>  /** Base turn rate, in deg/sec. Other scaling may affect final turn rate. */
</strong><strong>  UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category=Camera)
</strong><strong>  float BaseTurnRate;
</strong><strong>  
</strong><strong>  /** Base look up/down rate, in deg/sec. Other scaling may affect final rate. */
</strong><strong>  UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category=Camera)
</strong><strong>  float BaseLookUpRate;
</strong><strong>  
</strong><strong>  //** Behavior Tree for an AI Controller (Added in Chapter 2)
</strong><strong>  UPROPERTY(EditAnywhere, BlueprintReadWrite, category=AI)
</strong><strong>  
</strong><strong>  UBehaviorTree* BehaviorTree;
</strong></code></pre>

此外，要编译前述代码，我们还需要在类的顶部包含以下语句，就在 `.generated` 之前：

```cpp
#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "BehaviorTree/BehaviorTree.h"
#include "UnrealAIBookCharacter.generated.h"
```

关闭 `Character` 类，因为我们已经完成了对它的处理。因此，每当我们在世界中放置该角色实例时，我们将能够通过详细信息面板指定一个行为树，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption></figcaption></figure>

让我们打开我们刚刚创建的AI控制器的头文件（如果你使用的是Visual Studio作为IDE，它应该已经打开了）。特别是，我们需要重写AI控制器类的一个函数。我们要重写的函数叫做Possess()，它允许我们在AI控制器控制一个新的Pawn（也就是角色，是一个Pawn）时运行一些代码。在保护可见性内添加以下代码（用粗体表示）：

```cpp
UCLASS()
class UNREALAIBOOK_API AMyFirstAIController : public AAIController
{
    GENERATED_BODY()

protected:

    /** override the OnPossess function to run the behavior tree.
    void OnPossess (APawn* InPawn) override;

};
```

接下来，打开实现（.cpp）文件。同样，要使用行为树，我们必须包含Behavior Trees和UnrealAIBookCharacter类：

```cpp
#include "MyFirstAIController.h"
#include "UnrealAIBOokCharacter.h"
#include "BehaviorTree/BehaviorTree.h"
```

接下来，我们需要为Possess()函数分配一个功能。我们需要检查Pawn是否真的是一个**UnrealAIBookCharacter**，如果是，我们就获取行为树并运行它。当然，这被一个if语句包围着，以避免我们的指针为空：

```cpp
void AMyFirstAIController::OnPossess(APawn* InPawn)
{
    Super::OnPossess(InPawn);
    AUnrealAIBookCharacter* Character = Cast<AUnrealAIBookCharacter>(InPawn);
    if (Character != nullptr)
    {
        UBehaviorTree* BehaviorTree = Character->BehaviorTree;
        if (BehaviorTree != nullptr) {
            RunBehaviorTree(BehaviorTree);
        }
    }
}
```

{% hint style="info" %}
如果由于任何原因你无法让代码运行，你可以直接使用蓝图控制器来启动行为树，或者继承C++控制器，确保其余的代码都能运行，并在蓝图中调用RunBehaviorTree()函数。
{% endhint %}

一旦我们编译了项目，我们将能够使用这个控制器。从关卡中选择我们的AI角色（如果你没有它，可以创建一个），然后在详情面板中，我们可以设置我们的C++控制器，如下所示：

<figure><img src="../../../.gitbook/assets/image (25) (1) (1).png" alt=""><figcaption></figcaption></figure>

此外，不要忘记分配行为树，我们总是在详情面板中进行此操作：

<figure><img src="../../../.gitbook/assets/image (26) (1) (1).png" alt=""><figcaption></figcaption></figure>

因此，一旦游戏开始，敌人将开始执行行为树。目前，树是空的，但这为我们提供了开始使用行为树所需的结构。在接下来的章节中，我们将更详细地探索行为树，特别是在第8、9和10章中，我们将查看更实用的设计和构建行为树的方法。
