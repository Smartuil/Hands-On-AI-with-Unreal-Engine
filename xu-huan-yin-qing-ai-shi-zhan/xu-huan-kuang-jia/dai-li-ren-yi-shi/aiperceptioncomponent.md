# AIPerceptionComponent

让我们分解一下AIPerceptionComponent的工作原理。我们将同时在蓝图和C++中进行讲解。

**蓝图中的AIPerceptionComponent**

如果我们打开我们的蓝图AI控制器，我们可以像添加其他组件一样添加AIPerceptionComponent：从组件选项卡中，点击添加组件并选择AIPerceptionComponent，如下图所示：

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

选择组件后，您将看到它在详情面板中的显示方式，如下图所示：

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

它只有两个参数。一个定义了主导感官。实际上， AIPerceptionComponent可以拥有多种感官，当需要检索被感知目标的位置时，AI应该使用哪一个呢？主导感官通过给予一种感官优先权来消除歧义。另一个参数是感官数组。当你用不同的感官填充数组时，你将能够自定义每一个感官，如下图所示：

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
请记住，你可以拥有多种类型的感官。 假设你的敌人有两个头，面向不同的方向：你可能想要有两个视觉感官，每个头一个。当然，在这种情况下，需要更多的设置才能使它们正确工作，因为你需要修改视觉组件的工作方式，比如说，AI总是从其前向量观察。
{% endhint %}

&#x20;每种感官都有其自己的属性和参数。让我们来看看两个主要的：视觉和听觉。

**感知 - 视觉**

视觉感知的工作原理如你所期望的那样，而且它几乎是现成的（对于其他感官来说可能并非如此，但视觉和听觉是最常见的）。下面是它的外观：&#x20;

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

让我们分解一下控制视觉感知的主要参数：&#x20;

* 视觉半径：如果目标（一个可见的物体）进入这个范围，并且没有被遮挡，那么目标就会被探测到。从这个意义上说，这是“注意到目标的最大视觉距离”。
* 失去视觉半径：如果目标已经被看到，那么在这个范围内，如果目标没有被遮挡，它仍然会被看到。这个值大于视觉半径，意味着如果AI已经看到了目标，它能感知到目标的距离更远。从这个意义上说，这是“注意到已经被看到的目标的最大视觉距离”。
* 周边视觉半角度数：顾名思义，它指定了AI能看多远（以度数表示）。90度的值意味着（因为这个值只是角度的一半），AI能够看到前方180度范围内的所有东西。180度的值意味着AI可以向任何方向看；它有360度的视野。此外，需要注意的是，这个半角是从前向量测量的。
* 下图说明了这一点：

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

* 自动成功范围从最后出现：默认情况下，这被设置为一个无效值（-1.0f），意味着它不被使用。这指定了从目标最后出现位置的一个范围，如果目标在这个范围内，那么目标始终可见。

还有其他更通用的设置，可以应用于许多感知（包括听觉，因此它们不会在下一节中重复）：

* 根据隶属关系检测：参见不同团队部分。
* 调试颜色：顾名思义，这是该感知在视觉调试器中显示的颜色（更多信息，请参见第11章，AI调试方法 - 记录）。
* 最大年龄：它表示记录刺激的时间（以秒为单位）。想象一下，目标从AI的视线中消失；它的最后位置仍然被记录，并且它被分配了一个年龄（那个数据有多旧）。如果年龄超过了最大年龄，那么刺激就会被擦除。例如，AI正在追逐玩家，玩家从其视线中逃脱。现在，AI应该首先检查玩家最后出现的位置，试图将其重新带回视线。如果失败，或者位置是很久以前记录的，那么这些数据就不再相关，可以被擦除。总之，这指定了在由这种感知生成的刺激被遗忘后的年龄限制。此外，值0意味着永远不会。

**感官 - 听觉**

听觉只有一个适当的参数，即听觉范围。这设置了AI能够听到的距离。其他的参数我们已经看过的视觉通用参数一样（例如：最大年龄、调试颜色和根据隶属关系探测）。它看起来是这样的：&#x20;

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
为了使这本书完整，值得一提的是还有另一个选项，称为LoSHearing。据我所知，通过查看虚幻源代码（版本4.20），这个参数似乎不影响任何东西（除了调试）。因此，我们将其保留为不启用。&#x20;
{% endhint %}

无论如何，还有其他选项可以控制声音的产生。实际上，需要使用特殊功能/蓝图节点手动触发听觉事件。 **AIPerceptionComponent和C++中的感官**

如果您跳过了前面的部分，请先阅读它们。实际上，所有的概念都是相同的，在本节中，我只是在C++中展示该组件的用法，而不重新解释所有的概念。&#x20;

这是我们想要使用的类的#include语句（我只包括了视觉和听觉）：

{% code lineNumbers="true" %}
```cpp
#include "Perception/AIPerceptionComponent.h"
#include "Perception/AISense_Sight.h"
#include "Perception/AISenseConfig_Sight.h"
#include "Perception/AISense_Hearing.h"
#include "Perception/AISenseConfig_Hearing.h"
```
{% endcode %}

要将组件添加到C++ AI控制器中，就像添加其他组件一样，在.h文件中使用一个变量。因此，要跟踪它，您可以使用基类中的inerith变量，其声明如下：

{% code lineNumbers="true" %}
```cpp
UPROPERTY(VisibleDefaultsOnly, Category = AI)
UAIPerceptionComponent* PerceptionComponent;
```
{% endcode %}

{% hint style="info" %}
因此，您可以在任何AIController中使用此变量，而无需在头文件（.h）中声明它。
{% endhint %}

然后，在.cpp文件的构造函数中使用CreateDefaultSubobject()函数，我们可以创建组件：

{% code lineNumbers="true" %}
```cpp
PerceptionComponent =
CreateDefaultSubobject<UAIPerceptionComponent>(TEXT("SightPerceptionCompone
nt"));
```
{% endcode %}

此外，您将需要额外的变量，每个要配置的感官一个。 例如，对于视觉和听觉，您将需要以下变量：

{% code lineNumbers="true" %}
```cpp
UAISenseConfig_Sight* SightConfig;
UAISenseConfig_Hearing* HearingConfig;
```
{% endcode %}

要配置一个感知，你需要先创建它，你可以访问它的所有属性，并且可以设置你需要的：

{% code lineNumbers="true" %}
```cpp
//Create the Senses
SightConfig = CreateDefaultSubobject<UAISenseConfig_Sight>(FName("Sight
Config"));
HearingConfig =
CreateDefaultSubobject<UAISenseConfig_Hearing>(FName("Hearing Config"));
//Configuring the Sight Sense
SightConfig->SightRadius = 600;
SightConfig->LoseSightRadius = 700;
//Configuration of the Hearing Sense
HearingConfig->HearingRange = 900;
```
{% endcode %}

最后，你需要将感官绑定到AIPerceptionComponent上：

{% code lineNumbers="true" %}
```cpp
//Assigning the Sight and Hearing Sense to the AI Perception Component
PerceptionComponent->ConfigureSense(*SightConfig);
PerceptionComponent->ConfigureSense(*HearingConfig);
PerceptionComponent->SetDominantSense(SightConfig->GetSenseImplementation());
```
{% endcode %}

如果您需要回拨事件，可以通过绕过回调函数来做到这一点（它必须具有相同的签名，不一定具有相同的名称）：

{% code lineNumbers="true" %}
```cpp
//Binding the OnTargetPerceptionUpdate function
PerceptionComponent->OnTargetPerceptionUpdated.AddDynamic(this,
&ASightAIController::OnTargetPerceptionUpdate);
```
{% endcode %}

这结束了在C++中使用AIPerceptionComponent和Senses。
