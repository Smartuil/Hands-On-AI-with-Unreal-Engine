# 实际操作感知系统 - 视觉AI控制器

我们了解某样东西的最好方法就是使用它。因此，让我们从创建一个简单的感知系统开始，在该系统中，当有物体进入或离开AI的感知范围时，我们会在屏幕上打印出来，同时显示当前看到的物体数量（包括/不包括刚进入/离开的那个物体）。&#x20;

我们将再次这样做两次，一次使用蓝图，另一次使用C++，这样我们就可以了解两种创建方法。&#x20;

**蓝图感知系统**

首先，我们需要创建一个新的AI控制器（除非你想使用我们已经使用过的那个）。在这个例子中，我将其称为"SightAIController"。打开蓝图编辑器，添加AIPerception组件，并随意重命名它（如果你喜欢的话），比如叫做"SightPerceptionComponent"。&#x20;

选择这个组件。在细节面板中，我们需要将其作为视觉感知添加进去，如下图所示：

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

我们可以将视野半径和失去视野半径设置为合理的值，例如分别为600和700，这样我们就有了这样的设置：

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

我们可以保持角度不变，但需要更改检测方式。实际上，无法从蓝图更改团队，因此玩家将处于相同的第 255 中立团队。由于我们刚开始研究系统的工作原理，我们可以勾选全部三个复选框。现在，我们应该得到这样的结果：

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

在组件的底部，我们应该有所有不同的事件。特别是，我们需要**On Target Perception Updated**，每当目标进入或退出感知区域时都会调用它——这正是我们所需要的：

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

点击"+"号在图表中添加事件：

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

此事件将为我们提供导致更新并创建刺激的Actor（值得记住的是，感知组件在那时可能有多个感知，此变量告诉你哪个刺激导致了更新）。在我们的案例中，我们只有视觉，所以它不可能是其他的。下一步是理解我们有多少目标洞察，以及哪个目标离开了视野或进入了视野。&#x20;

所以，将**SightPerceptionComponent**拖入图表。从那里，我们可以拖动一个引脚来获取所有“**当前感知到的Actor**”，这将返回一个Actor数组。别忘了将感知类别设置为视觉：

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

通过测量这个数组的长度，我们可以得到当前感知到的角色数量。此外，通过检查从事件中传递过来的角色是否在当前“看到的角色”数组中，我们可以判断这样的角色是离开了视野还是进入了视野：

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

最后一步是将所有这些信息格式化为一个漂亮的格式化字符串，以便可以在屏幕上显示。我们将使用Append节点来构建字符串，以及一个选择“进入”或“离开”的Actor。最后，我们将最终结果插入到Print String中。

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
打印字符串仅用于调试目的，在发布游戏时不可用，但我们只是在测试和理解感知系统的工作原理。另外，我知道当感知到的角色数量为一时，字符串会产生“1 objects”，这是不正确的，但纠正复数形式（尽管可能，无论是使用if语句还是以更复杂的方式处理语言结构）超出了本书的范围。这就是为什么我使用这个表达式。
{% endhint %}

保存AI控制器并返回关卡。如果你不想用C++做同样的事情，跳过下一节，直接转到“全面测试”。

**C++感知系统**

如果你更偏向C++，或者想尝试如何用C++构建相同的AI控制器，这就是为你准备的部分。我们将遵循完全相同的步骤（或多或少），而不是图像，我们将有代码！&#x20;

让我们从创建一个新的AIController类开始（如果你不记得如何做，请参阅第2章，在AI世界迈出第一步）。我们将其命名为SightAIController，并将其放在AIControllers文件夹内。&#x20;

让我们开始编辑SightAIController.h文件，在该文件中，我们需要包含一些其他的.h文件，以便我们的编译器知道我们需要的类的实现在哪里。实际上，我们将需要访问AIPerception和AISense\_Config类。因此，在代码文件的顶部，你应该有以下#include语句：

{% code lineNumbers="true" %}
```cpp
#pragma once
#include "CoreMinimal.h"
#include "AIController.h"
#include "Perception/AIPerceptionComponent.h"
#include "Perception/AISense_Sight.h"
#include "Perception/AISenseConfig_Sight.h"
#include "SightAIController.generated.h"
```
{% endcode %}

然后，在我们的类中，我们需要保持对AIPerception组件的引用以及一个额外的变量来保存Sight感知的配置：

{% code lineNumbers="true" %}
```cpp
//Components Variables
 UAIPerceptionComponent* PerceptionComponent;
 UAISenseConfig_Sight* SightConfig;
```
{% endcode %}

此外，我们需要添加构造函数，以及OnTargetPerceptionUpdate事件的回调。为了使其工作，最后一个必须是UFUNCTION()，并且需要以Actor和AIStimulus作为输入。这样，反射系统将按预期工作：

{% code lineNumbers="true" %}
```cpp
//Constructor
ASightAIController();
//Binding function
UFUNCTION()
void OnTargetPerceptionUpdate(AActor* Actor, FAIStimulus Stimulus);
```
{% endcode %}

让我们进入.cpp文件。首先，我们需要创建AIPerception组件，以及一个视觉配置：

{% code lineNumbers="true" %}
```cpp
ASightAIController::ASightAIController() {
  //Creating the AI Perception Component
  PerceptionComponent =
  CreateDefaultSubobject<UAIPerceptionComponent>(TEXT("SightPerceptionComponent"));
  SightConfig = CreateDefaultSubobject<UAISenseConfig_Sight>(FName("SightConfig"));
}
```
{% endcode %}

然后我们可以用相同的参数配置视觉感应：视野半径设为600，失去视野半径设为700：

{% code lineNumbers="true" %}
```cpp
ASightAIController::ASightAIController() {
  //Creating the AI Perception Component
  PerceptionComponent =
  CreateDefaultSubobject<UAIPerceptionComponent>(TEXT("SightPerceptionComponent"));
  SightConfig = CreateDefaultSubobject<UAISenseConfig_Sight>(FName("SightConfig"));
  //Configuring the Sight Sense
  SightConfig->SightRadius = 600;
  SightConfig->LoseSightRadius = 700;
}
```
{% endcode %}

接下来，我们需要检查DetectionByAffiliation的所有标志，以便我们检测到我们的玩家（因为，目前，他们都在第255队；查看练习部分以了解如何改进这一点）：

<pre class="language-cpp" data-line-numbers><code class="lang-cpp">ASightAIController::ASightAIController() {
  //Creating the AI Perception Component
  PerceptionComponent =
<strong>  CreateDefaultSubobject&#x3C;UAIPerceptionComponent>(TEXT("SightPerceptionComponent"));
</strong>  SightConfig = CreateDefaultSubobject&#x3C;UAISenseConfig_Sight>(FName("SightConfig"));
  //Configuring the Sight Sense
  SightConfig->SightRadius = 600;
  SightConfig->LoseSightRadius = 700;
  SightConfig->DetectionByAffiliation.bDetectEnemies = true;
  SightConfig->DetectionByAffiliation.bDetectNeutrals = true;
  SightConfig->DetectionByAffiliation.bDetectFriendlies = true;
}
</code></pre>

最后，我们将视觉配置与AIPerception组件关联，并将OnTargetPerceptionUpdate函数绑定到AIPerception组件上的同名事件。

{% code lineNumbers="true" %}
```cpp
ASightAIController::ASightAIController() {
  //Creating the AI Perception Component
  PerceptionComponent =
  CreateDefaultSubobject<UAIPerceptionComponent>(TEXT("SightPerceptionComponent"));
  SightConfig = CreateDefaultSubobject<UAISenseConfig_Sight>(FName("SightConfig"));
  //Configuring the Sight Sense
  SightConfig->SightRadius = 600;
  SightConfig->LoseSightRadius = 700;
  SightConfig->DetectionByAffiliation.bDetectEnemies = true;
  SightConfig->DetectionByAffiliation.bDetectNeutrals = true;
  SightConfig->DetectionByAffiliation.bDetectFriendlies = true;
  //Assigning the Sight Sense to the AI Perception Component
  PerceptionComponent->ConfigureSense(*SightConfig);
  PerceptionComponent->SetDominantSense(SightConfig->GetSenseImplementation()
);
  //Binding the OnTargetPerceptionUpdate function
  PerceptionComponent->OnTargetPerceptionUpdated.AddDynamic(this, &ASightAIController::OnTargetPerceptionUpdate);
}
```
{% endcode %}

这结束了构造函数，但我们仍需要实现OnTargetPerceptionUpdate()函数。首先，我们需要检索所有当前感知到的Actor。这个函数需要一个它可以填充的Actor数组，以及要使用的感知的实现。

因此，我们的数组将填充满感知到的角色：

{% code lineNumbers="true" %}
```cpp
void ASightAIController::OnTargetPerceptionUpdate(AActor* Actor,
FAIStimulus Stimulus)
{
  //Retrieving Perceived Actors
  TArray<AActor*> PerceivedActors;
  PerceptionComponent->GetPerceivedActors(TSubclassOf<UAISense_Sight>(), PerceivedActors);
}
```
{% endcode %}

通过测量这个数组的长度，我们可以得到当前感知到的角色数量。此外，通过检查从事件传递过来的角色（函数的参数）是否在当前“看到的角色”数组中，我们可以判断这样的角色是离开了视野还是进入了视野：

{% code lineNumbers="true" %}
```cpp
void ASightAIController::OnTargetPerceptionUpdate(AActor* Actor,
FAIStimulus Stimulus)
{
  //Retrieving Perceived Actors
  TArray<AActor*> PerceivedActors;
  PerceptionComponent->GetPerceivedActors(TSubclassOf<UAISense_Sight>(), PerceivedActors);
  //Calculating the Number of Perceived Actors and if the current target Left or Entered the field of view.
  bool isEntered = PerceivedActors.Contains(Actor);
  int NumberObjectSeen = PerceivedActors.Num();
}
```
{% endcode %}

最后，我们需要将这些信息打包成一个格式化的字符串，然后打印在屏幕上：

{% code lineNumbers="true" %}
```cpp
void ASightAIController::OnTargetPerceptionUpdate(AActor* Actor, FAIStimulus Stimulus)
{
  //Retrieving Perceived Actors
  TArray<AActor*> PerceivedActors;
  PerceptionComponent->GetPerceivedActors(TSubclassOf<UAISense_Sight>(), PerceivedActors);
  //Calculating the Number of Perceived Actors and if the current target Left or Entered the field of view.
  bool isEntered = PerceivedActors.Contains(Actor);
  int NumberObjectSeen = PerceivedActors.Num();
  //Formatting the string and printing it
  FString text = FString(Actor->GetName() + " has just " + (isEntered ? "Entered" : "Left") + " the field of view. Now " + FString::FromInt(NumberObjectSeen) + " objects are visible.");
  if (GEngine) {
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Turquoise, text);
  }
  UE_LOG(LogTemp, Warning, TEXT("%s"), *text);
}
```
{% endcode %}

{% hint style="info" %}
再一次，我知道"1 objects"是不正确的，但更正复数形式（虽然可能）超出了本书的范围；让我们保持简单。
{% endhint %}

**测试全部**

现在，您应该拥有一个实现了感知系统的AI控制器（无论是用蓝图还是C++实现的 - 没关系，它们的行为应该完全相同）。&#x20;

通过按住Alt键并拖动玩家到关卡中，创建另一个第三人称角色（如果您想使用前面章节中创建的AI，可以这样做）：

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

在详情面板中，我们让它受我们的AI控制器控制，而不是玩家（这对你来说应该已经是一个容易的过程）：

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

或者，如果您使用的是C++设置，请选择以下设置：

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

在按下播放之前，创建一些可以被检测的其他对象会很好。我们知道所有的Pawn都会被检测到（除非被禁用），所以让我们尝试一些不是Pawn的东西——也许是一个移动的平台。因此，如果我们想要检测它，我们需要使用**AIPerceptionStimuliSourceComponent**。&#x20;

首先，让我们创建一个浮动平台（我们的角色可以轻松推动它）。如果你在ThirdPersonCharacter Example的默认关卡中，你可以通过Alt + 拖动来复制这个大的网格，如下截图中高亮的部分（否则，如果你使用的是自定义关卡，一个可以被压扁的立方体也可以很好地工作）：

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

到目前为止，它太大了，让我们将其缩小到(1, 1, 0.5)。同时，为了达成共识，你可以将其移动到(-500, 310, 190)。最后，我们需要将可移动性更改为可移动，因为它需要移动：

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

接下来，我们希望推出这样一个平台，因此我们需要启用物理模拟。为了让角色能够推动它，我们给它一个100公斤的质量（我知道，看起来很多，但是由于摩擦力小，加上平台是浮动的，这是合适的量）。此外，我们不希望平台旋转，所以我们需要在约束中阻止所有三个旋转轴。如果我们想让平台浮动，也是如此——如果我们锁定z轴，平台只能沿XY平面移动，不能旋转。这将确保一个好的可推动的平台。这就是物理部分应该的样子：

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

最后，我们需要从Actor名称附近的**添加组件**绿色按钮添加一个**AIPerceptionStimuliSourceComponent**：

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

添加组件后，我们可以从前面的菜单中选择它。结果，详情面板将允许我们更改**AIPerceptionStimuliSourceComponent**设置。特别是，我们想添加视觉感知，并自动将该组件注册为源。我们应该这样设置：

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

作为可选步骤，你可以将其转换为蓝图以便重复使用，并且可以分配一个更有意义的名称。另外，如果你想让视线感知系统跟踪多个对象，你可以复制几份。

最后，你可以点击播放并测试我们迄今为止所取得的成果。如果你通过了我们的AI控制的角色，屏幕顶部会出现通知。如果我们将平台推入或推出AI的视野，我们也会得到相同的输出。在下面的截图中，你可以看到C++实现，但它与蓝图实现非常相似（只是打印的颜色变了）：

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

另外，作为预期，可以通过视觉调试器看到AI视野， 我们将在第13章“AI调试方法 - 游戏性调试器”中探讨。 以下截图是我们创建的AI角色的视野参考。有关如何显示它以及理解所有这些信息的含义的详细信息， 请继续阅读第13章“AI调试方法 - 游戏性调试器”：

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

是时候给自己点个赞了，因为看似只做了一点，但实际上你已经学会了复杂的系统。另外，如果你尝试了一种方式（蓝图或C++），如果你想掌握系统的蓝图和C++，可以尝试另一种方式。
