# 修改导航网格

到目前为止，我们已经了解了如何生成导航网格。然而，我们希望能够对其进行修改以更好地满足我们的需求。如前所述，可能存在一些穿越成本较高的区域，或者导航网格的两个点之间似乎被隔断（例如，通过一个台阶）的连接。因此，本节将探讨Unreal中用于修改导航网格以适应关卡的不同工具。

**导航修改器体积**

好了，现在是时候看看如何开始修改导航网格了。例如，导航网格的某一部分可能不希望被穿越，或者我们想要具有不同属性的另一部分。我们可以通过使用导航修改器体积来实现这一点。

你可以在模式面板的体积选项卡下找到这个设置，然后进入导航网格边界体积：

<figure><img src="../../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

一旦这个体被放置在地图中，默认值是移除该体内部的**导航网格**部分，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

这在您有AI不想进入的区域或修复导航网格的伪影时非常有用。尽管**导航修改器体**指定了地图的一部分，但行为是在**导航网格区域**中指定的。这意味着，如果我们查看**导航网格修改器体**的设置，我们只能找到与导航相关的一个设置，名为**区域类**：

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

因此，这个体只能指定地图中应用特定**区域类**的一部分。默认情况下，区域类是**NavArea\_Null**，它在地图与该体重叠的部分“移除”导航网格。我们将在下一节中探讨**导航网格区域**的工作原理。

**导航网格区域**

在前一节中，我们讨论了地图的可导航区域并非所有部分都是平等的。如果有一个区域被认为是危险的，AI 应该避开它。Unreal 的内置导航系统能够通过使用成本来处理这些不同区域。这意味着 AI 将通过汇总路径上所有的成本来评估要走的路径，并选择成本最低的那条。

此外，值得指出的是有两种类型的成本。对于每个区域，进入（或离开）区域的初始成本和穿越区域的成本。让我们看几个例子来澄清这两者之间的区别。

想象一下有一个森林，但在森林的每个入口处，AI 需要向生活在森林中的土著支付通行费。然而，一旦进入，AI 可以自由移动，就像他们在外面的森林一样。在这种情况下，进入森林是有成本的，但一旦进入，就没有成本需要支付。因此，当 AI 需要评估是否穿越森林时，这取决于是否有另一种方式可以去，以及这样做需要多长时间。

现在，想象一下有一个充满毒气的区域。在这个第二种情况下，进入该区域的成本可能是零，但穿越它的成本很高。实际上，AI 在该区域待的时间越长，失去的生命值就越多。是否值得进入不仅取决于是否有另一种方式，以及穿越那种替代方式需要多长时间（就像在前一种情况中一样），还取决于一旦进入，AI 需要穿越该区域多长时间。

在 Unreal 中，成本是在类内指定的。如果你点击一个 **Nav Modifier Volume**，你会注意到你需要指定一个 **Area Class**，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

正如你可能猜到的，默认值是 **NavArea\_Null**，它对进入的成本是无限的，导致 AI 从不进入该区域。导航系统足够智能，甚至不会去生成那个区域，并将其视为不可导航的区域。

然而，你可以改变**区域类别**。默认情况下，你将能够访问以下区域类别：

* NavArea\_Default：这是生成的默认区域。如果你想在同一地点拥有多个这样的修饰符，将它作为修饰符是很有用的。
* NavArea\_LowHeight：这表示该区域并不适合所有代理，因为高度减少了（例如，在通风隧道的情况下，并非所有的代理都能适应/蹲下）。
* NavArea\_Null：这使得该区域对所有代理都不可导航。
* NavArea\_Obstacle：这为该区域分配了更高的成本，因此代理会试图避开它：

<figure><img src="../../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
你会注意到，如果你创建一个新的蓝图，或者甚至在 Visual Studio 中打开源代码，都会有 NavArea\_Meta 和它的子类 NavArea\_MetaSwitchingActor。然而，如果你看它们的代码，它们主要包含一些已经废弃的代码。因此，我们在这本书中不会使用它们。
{% endhint %}

然而，你可以通过扩展 **NavArea 类**来扩展不同区域的列表（并可能增加更多功能）。让我们看看如何在蓝图和 C++ 中实现这一点。当然，正如我们在前一章中所做的，我们将创建一个名为 Chapter3/Navigation 的新文件夹，我们将在其中放置我们所有的代码。

**在蓝图中的NavArea类创建**

在蓝图中创建一个新的**NavArea**类非常简单；你只需要创建一个新的蓝图，该蓝图继承自**NavArea**类，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

根据约定，类名应以“_NavArea\__”开头。我们将其重命名为**NavArea\_BPJungle**（我添加了BP以示我们是用蓝图创建的，因为我们在蓝图和C++中重复了相同的任务）。这在内容浏览器中应该是这样的：

<figure><img src="../../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

然后，如果你打开蓝图，你将能够为区域分配自定义成本。你也可以为你的区域指定特定的颜色，以便在构建导航网格时容易识别。这是默认情况下详情面板的样子：

<figure><img src="../../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

现在，我们可以根据需要自定义。例如，我们可能希望进入丛林有一定的成本，穿越丛林的成本稍高。我们将使用亮绿色作为颜色，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

一旦编译并保存，我们可以将这个新创建的区域分配给**导航修改器体积**，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

这就是我们的成品类在我们的关卡中看起来的样子（如果导航网格可见）：

<figure><img src="../../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

**创建C++中的NavArea类**

在C++中创建**NavArea**类也很简单。首先，你需要创建一个新的C++类，该类继承自**NaoArea**类，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

根据约定，名称应以“_NavArea\__”开头。因此，您可以将其重命名为**NavArea\_Desert**（只是为了改变AI可能面临的地形种类，因为我们之前创建了一个丛林）并将其放在“**Chapter3/Navigation**”中：

<figure><img src="../../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

一旦创建了该类，您只需要在构造函数中分配参数。为了方便起见，这里是我们声明一个简单构造函数的类定义：

{% code lineNumbers="true" %}
```cpp
#include "CoreMinimal.h"
#include "NavAreas/NavArea.h"
#include "NavArea_Desert.generated.h"

UCLASS()
class UNREALAIBOOK_API UNavArea_Desert : public UNavArea
{
    GENERATED_BODY()

    UNavArea_Desert();
};
```
{% endcode %}

然后，在构造函数的实现中，我们可以分配不同的参数。例如，我们可以为进入设置较高的成本，并为穿越设置更高的成本（相对于默认或丛林）。此外，我们可以将颜色设置为黄色，以便我们记住这是一个沙漠区域：

{% code lineNumbers="true" %}
```cpp
#include "NavArea_Desert.h"

UNavArea_Desert::UNavArea_Desert()
{
    DefaultCost = 1.5f;
    FixedAreaEnteringCost = 3.f;
    DrawColor = FColor::Yellow;
}
```
{% endcode %}

{% hint style="info" %}
你可以随时调整这些值，以找到最适合你的设置。例如，你可以创建一个进入成本非常高，但穿越成本很低的区域。因此，如果你只打算短暂穿越这个区域，那么它应该被避免；但是，如果代理长时间穿越它，可能比较短的路线更方便。
{% endhint %}

一旦创建了该类，可以将其设置为**导航修改器体积**的一部分，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

结果，你将能够在Nao网格中看到你的自定义区域（在这种情况下，使用黄色）：

<figure><img src="../../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

**导航链接代理**

默认情况下，如果有一个悬崖，AI不会掉下去，即使这可能是他们到达目的地的最短路径。实际上，悬崖顶部的导航网格与底部的Nao网格没有直接连接。然而，Unreal Navigation System提供了一种通过所谓的**导航链接代理**来连接Nav网格中任意两个三角形的方法。

{% hint style="info" %}
尽管区域是连接的，路径查找器会找到正确的道路，但AI不能违反游戏规则，无论是物理规则还是游戏机制。这意味着如果AI无法跳跃或穿越魔法墙，角色将会卡住，因为路径查找器返回了一条路径，但角色无法执行它。
{% endhint %}

让我们更详细地探索这个工具。

**创建导航链接代理**

要使用链接连接两个区域，我们需要转到“所有类别”选项卡中的“模式”面板，并选择“导航链接代理”，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
或者，你可以在“模式”面板中搜索它，以便更快地找到它：

![](<../../../.gitbook/assets/image (198).png>)
{% endhint %}

一旦链接被放置在关卡中，你会看到一个“箭头/链接”，你将能够修改链接的起点和终点。它们被称为“**左**”和“**右**”，设置它们位置的最简单方法是通过拖放（和放置）它们在视口中。因此，你将能够连接导航网格的两个不同部分。正如我们在下面的截图中看到的，如果导航网格可见（使用P键启用），你会看到一个箭头连接“**右**”和“**左**”节点。这个箭头指向两个方向。这将导致链接成为双向的：

<figure><img src="../../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
你可能注意到有两个箭头，其中一个颜色较深的绿色。此外，这个第二个箭头/链接可能并不完全在你放置右端的位置，而是附着在Nav Mesh上。你可以在下面的截图中更清楚地看到这个第二个箭头：

![](<../../../.gitbook/assets/image (200).png>)

这实际上是Nav Mesh连接的方式，由于链接的投影设置。我们将在下一节中探讨这个设置。
{% endhint %}

如果你想让链接只朝一个方向移动，我们可以在详情面板中更改这个设置。然而，要探索这些设置，我们首先需要理解有两种不同类型的链接：**简单链接**和**智能链接**。

**简单链接和智能链接**

当我们创建一个**导航链接代理**时，它会附带一个简单链接数组。这意味着，通过一个**导航链接代理**，我们可以将导航网格的不同部分连接在一起。然而，**导航链接代理**也会附带一个**智能链接**，默认情况下是禁用的。

让我们学习一下**简单链接**和**智能链接**之间的相似性和区别。

**简单链接和智能链接**

**简单链接和智能链接**的行为方式相似，它们都将导航网格的两个部分连接在一起。此外，这两种类型的链接都可以有**方向**（从左到右、从右到左或双向）和**导航区域**（链接所在的导航区域类型；例如，您可能希望在使用的链接时具有自定义成本）。

**简单链接**

简单链接存在于导航代理链接内的**点链接数组**中，这意味着在单个导航代理链接中可以有多条简单链接。要创建另一个简单链接，您可以向详细信息面板中的简单节点数组添加一个额外元素，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

一旦我们有了更多的简单链接，我们可以设置**开始**和**结束**位置，就像我们对第一个链接所做的那样（通过选择它们并在视口中移动它们，就像对其他任何演员一样）。下面的截图显示了我将两个简单链接放置在同一个导航代理链接上，彼此相邻：

<figure><img src="../../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
每次我们创建一个 _Nao Link 代理_时，它都会附带一个 _Simple Link_ 数组。
{% endhint %}

对于 Point Links 数组中的每个 **Simple Link**，我们可以通过展开该项来访问其设置。以下截图显示了第一个 Simple Link 的设置：

<figure><img src="../../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

让我们理解这些各种设置：

* **左端和右端**：链接的左端和右端的位置。
* **左投影高度和右投影高度**：如果这个数值大于零，那么链接将被投影到导航几何体（使用由该数值指定的最大长度的跟踪）的左端和右端。你可以在下面的截图中看到这个投影的链接：

<figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

* **方向**：这指定了链接工作的方向。此外，视口中的箭头也会相应更新。可能的选项如下：
  * **双向**：链接是双向的（请记住，AI需要配备在两个方向上穿越链接的能力；例如，如果我们正在越过一个悬崖，代理需要能够从它上面掉下来（链接的一个方向）并跳跃（链接的另一个方向）。
  * **从左到右**：链接只能从左端穿越到右端（代理仍然需要有进入该链接方向的能力）。
  * **从右到左**：链接只能从右端到左端跨越（代理仍然需要具有进入该链接方向的能力）。
* **贴紧半径和高度半径**：您可能已经注意到一个连接每条链接末端的圆柱体。这两个设置控制该圆柱体的半径和高度。有关此圆柱体的使用的更多信息，请检查“贴紧最便宜区域”。以下截图显示第一个链接有一个更大的圆柱体（半径和高度都更大）：

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

* **描述**：这只是您可以插入方便描述的字符串；它对导航或链接没有影响。
* **贴紧最便宜区域**：如果启用，它将尝试将链接末端连接到由贴紧半径和高度半径指定的圆柱体内可用的三角形中最便宜的区域。例如，如果圆柱体与默认导航区域和BP丛林导航区域（我们之前创建的）相交，链接将直接连接到默认导航区域，而不是丛林。
* **区域类别**：链接可能具有穿越成本，或者是特定导航区域。此参数允许您定义链接在穿越时是哪种类型的导航区域。

这就结束了简单链接的所有可能性。然而，这是一个非常强大的工具，可以让你塑造 Nao Mesh 并实现惊人的 AI 行为。现在，让我们深入探讨智能链接。

**智能链接**

智能链接可以在运行时通过“**Smart Link Is Relevant**”布尔变量启用和禁用。你还可以通知周围的actor这种变化。默认情况下，它是不相关的（也就是说，链接不可用），每个 Nav Proxy 链接只有一个**智能链接**。

{% hint style="info" %}
**请注意，不要混淆**：智能链接可以处于两种状态：启用和禁用。然而，如果链接实际上“存在/存在”（对于导航网格），那是另一个属性（Smart Link Is Relevant），换句话说，这意味着链接对导航系统“活跃”（但它仍然可以处于启用或禁用状态）。
{% endhint %}

不幸的是（至少对于当前版本的引擎），这些在编辑器中不可见，这意味着**起始**和**结束**位置需要手动设置。

然而，让我们看一下智能链接的设置：

<figure><img src="../../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

* **启用区域类别**：这是链接启用时假设的 Nao 区域。默认是 NavArea\_Default。
* **禁用区域类别**：这是链接禁用时假设的 Nav 区域。这意味着，当链接禁用时，如果分配了可穿越的区域，它仍然可以被穿越（例如，当链接禁用时，我们可能希望穿越的成本非常高，但我们仍然希望它能够被穿越。当然，默认是 NavArea\_Default，这意味着它是不可穿越的）。
* **链接相对起点**：这表示链接的起点，相对于其导航链接代理的位置。
* **链接相对终点**：这表示链接的终点，相对于其导航链接代理的位置。
* **链接方向**：这指定了链接的工作方向。可能的选择如下：
  * **双向**：链接是双向的（请记住，AI需要配备在两个方向上穿越链接的能力；例如，越过一个边缘，代理需要能够从它上面掉下来（链接的一个方向）和跳跃（链接的另一个方向））。
  * **从左到右**：链接只能从左端穿越到右端（代理仍然需要具有在该链接方向上移动的能力）。
  * **从右到左**：链接只能从右端穿越到左端（代理仍然需要具有在该链接方向上移动的能力）。

{% hint style="info" %}
尽管这个参数的选项将链接的终点标记为左和右，但它们指的是链接的起点和终点。或者（这可能更好，因为链接可以是双向的），链接相对起点和链接相对终点指的是左和右。
{% endhint %}

* **链接启用**：这是一个布尔变量，用于确定智能链接是否启用。这个值可以在运行时更改，链接可以“通知”周围对这类信息感兴趣的代理/演员（更多信息参见后面）。默认值是true。
* **智能链接是否相关**：这是一个布尔变量，用于确定智能链接是否“活跃”，即它是否相关或者我们应该忽略它。默认值是false。

这些是关于智能链接的主要设置。

值得一提的是，智能链接实际上可以做的不仅仅是连接导航网格。它们有一系列处理正在穿越链接的代理的功能。例如，通过打开NavLinkProxy.h文件，我们可以找到以下函数：

{% code lineNumbers="true" %}
```cpp
/** 代理在路径跟踪过程中到达智能链接时调用，使用ResumePathFollowing()来恢复控制 /
UFUNCTION(BlueprintImplementableEvent)
void ReceiveSmartLinkReached(AActor Agent, const FVector& Destination);

/** 恢复正常的路径跟踪 */
UFUNCTION(BlueprintCallable, Category="AI|Navigation")
void ResumePathFollowing(AActor* Agent);

/** 检查智能链接是否启用 */
UFUNCTION(BlueprintCallable, Category="AI|Navigation")
bool IsSmartLinkEnabled() const;

/** 改变智能链接的状态 */
UFUNCTION(BlueprintCallable, Category="AI|Navigation")
void SetSmartLinkEnabled(bool bEnabled);

/** 检查是否有任何代理正在通过智能链接移动 */
UFUNCTION(BlueprintCallable, Category="AI|Navigation")
bool HasMovingAgents() const;
```
{% endcode %}

不幸的是，这些函数在这本书的范围内无法详细讲解，但我邀请你阅读代码以了解更多信息。

之前我们提到，智能链接可以在运行时向其附近的代理/演员广播其状态变化的信息。你可以通过广播设置来改变智能链接如何广播这些信息，这些设置就在智能链接设置的下方：

<figure><img src="../../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

这些设置相当直观，但让我们快速浏览一下：

* **启用时通知**：如果为真，当链接启用时，它将通知代理/演员。
* **禁用时通知**：如果为真，当链接禁用时，它将通知代理/演员。
* **广播半径**：这指定了广播的范围。在这个半径之外的每个代理将不会收到链接状态变化的通知。
* **广播间隔**：这指定了链接应该每隔多久重复广播一次。如果值为零，广播只会重复一次。
* **广播通道**：这是用于广播变化的跟踪通道。

这结束了我们对智能链接的讨论。

**导航链接代理的其他设置**

最后，值得一提的是，当生成导航网格时，导航链接代理可以创建一个障碍盒。你可以在导航链接代理的详细信息面板中找到这些设置，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

这些设置允许你决定障碍盒是否激活/使用，它的尺寸/范围及其偏移量，以及导航区域的类型。

**扩展导航链接代理**

如果你想知道是否可以将链接扩展或将其包含在更复杂的演员中，答案是“当然可以！但你只能在C++中扩展它们。”

由于这本书无法涵盖所有内容，我们没有时间详细处理这个问题。然而，你可能想要扩展导航链接代理的一些原因是为了更好地控制进入链接的角色。例如，你可能想要一个跳跃垫将角色推过链接。这并不复杂，如果你在网上搜索，你会发现很多关于如何使用导航链接来实现这一点的教程。

只要记住，要成为Unreal中的优秀AI程序员，你最终需要掌握这部分导航链接，但就目前而言，我们涵盖的内容已经足够。

**导航回避**

导航回避是一个非常广泛的话题，Unreal有一些子系统为我们处理这个问题。因此，我们将在第6章“人群”中讨论这个主题。

**导航过滤**

我们并不希望每次都以相同的方式找到某条路径。想象一下，我们的AI代理人使用了一个增强道具，能够将穿越丛林的速度提高一倍。在这种情况下，导航系统并不会意识到这种变化，这也不是对Nav Mesh形状或权重的永久性改变。

导航过滤允许我们定义在特定时间段内如何执行路径查找的具体规则。你可能已经注意到，每次我们执行导航任务，无论是在蓝图还是C++中，都有一个可选参数用于插入导航过滤器。以下是一些具有此可选过滤参数的蓝图节点（C++函数也是如此）的示例：

<figure><img src="../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

甚至**行为树**中的“**移动到**”节点也有**导航过滤器**选项：

<figure><img src="../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

当然，一旦插入了过滤器，路径寻路将会相应地表现。这意味着使用**导航过滤器**非常简单。然而，我们如何在蓝图和C++中创建**导航过滤器**呢？让我们一起来了解一下。

**在蓝图中创建导航过滤器**

在本章的前面部分，我们在蓝图中创建了一个丛林区域。因此，这似乎是一个很好的例子，我们可以用它来创建一个**导航过滤器**，使AI代理能够更快地穿越丛林区域——甚至比穿越导航网格的默认区域还要快。让我们设想AI代理在关卡中的丛林类型区域具有某种能力或特性，使其能够更快地移动。

在蓝图中创建一个**导航过滤器**，我们需要开始创建一个新的蓝图，该蓝图继承自`NavigationQueryFilter`，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

根据惯例，类的名称应该以“NavFilter\_”开头。我们将其重命名为 NavFilter\_BPFastJungle（我添加了 BP，以便记住我是用蓝图创建这个的，因为我们在蓝图和 C++ 中重复同样的任务）。这在内容浏览器中应该如下所示：

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

一旦我们打开蓝图，我们可以在详细信息面板中找到其选项：

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

正如你所看到的，这里有一个**区域数组**和两套包含和排除（Nav）标志。不幸的是，我们并未涵盖导航标志，因为它们超出了本书的范围，并且只能在编写时通过C++进行分配。然而，**区域数组**相当有趣。让我们添加一个新的区域，并使用我们的**NavArea\_BPJungle**作为**区域类**，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

现在，我们可以覆盖丛林区域的**旅行成本**和**进入成本**，这些成本将替代我们在区域类中指定的成本，如果使用了此过滤器。请记住，勾选选项名称旁边的复选框以启用编辑。例如，我们可以将**旅行成本**设置为**0.6**（因为我们可以在丛林中快速移动而没有任何问题），并将**进入成本**设置为零：

<figure><img src="../../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

现在，我们已经准备好了。如果你更喜欢在丛林中旅行，过滤器已经可以使用！

{% hint style="info" %}
**改变导航区域的旅行成本并不会使AI代理在该区域移动得更快或更慢**，它只是使路径查找更倾向于选择那条路径。AI代理在该区域变得更快的实现被排除在导航系统之外，因此当AI角色在丛林中时，你需要实现这一点。
{% endhint %}

如果你也遵循了导航区域的C++部分，那么你的项目中也应该有沙漠区域。作为一个可选步骤，我们可以为过滤器添加第二个区域。想象一下，通过使用在丛林中更快移动的道具或能力，我们的角色对阳光非常敏感，非常容易晒伤，这显著降低了他们的健康值。因此，如果使用了这种过滤器，我们可以为沙漠区域设置更高的成本。只需添加另一个区域，并将**区域类别**设置为**NavArea\_Desert**。然后，覆盖成本；例如，**旅行成本**为**2.5**，**进入成本**为**10**：

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

一旦你完成了编辑设置，保存蓝图。从现在开始，你将能够在导航系统内使用这个过滤器。这就完成了如何在蓝图中创建**导航过滤器**的教程。

**创建C++中的导航过滤器**

与蓝图类似，我们可以在C++中创建一个**导航过滤器**。这次，我们可以创建一个稍微降低沙漠区域成本的过滤器。你可以在生活在沙漠中的某些动物上使用这个过滤器，使它们不那么容易受到其影响。

首先，我们需要创建一个新的C++类，该类继承自**NavigationQueryFilter**，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

根据约定，类名应以“_NavFilter\__”开头。因此，我们将其重命名为“**NavFilter\_DesertAnimal**”，并将其放置在“**Chapter3/Navigation**”中：

<figure><img src="../../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

为了设置其属性，我们需要创建一个默认构造函数。在头文件（.h）中编写以下内容：

{% code lineNumbers="true" %}
```cpp
#include "CoreMinimal.h"
#include "NavFilters/NavigationQueryFilter.h"
#include "NavFilter_DesertAnimal.generated.h"

/**
 *
 */
UCLASS()
class UNREALAIBOOK_API UNavFilter_DesertAnimal : public UNavigationQueryFilter
{
    GENERATED_BODY()

    UNavFilter_DesertAnimal();
};
```
{% endcode %}

对于实现（.cpp 文件），我们需要做更多的工作。首先，我们需要访问我们需要的 Nav Area，在这种情况下，就是沙漠。让我们添加以下 #include 语句：

{% code lineNumbers="true" %}
```cpp
#include "NavArea_Desert.h"
```
{% endcode %}

然后，在构造函数中，我们需要创建一个 **FNavigationFilterArea**，这是一个包含过滤特定类所有选项的类。在我们的例子中，我们可以将这个新的过滤区域存储在一个名为 Desert 的变量中：

{% code lineNumbers="true" %}
```cpp
UNavFilter_DesertAnimal::UNavFilter_DesertAnimal() {

    // 创建导航过滤区域
    FNavigationFilterArea Desert = FNavigationFilterArea();

    // [其余代码]
}
```
{% endcode %}

接下来，我们需要用我们想要为那个类覆盖的选项来填充 Desert 变量，包括我们正在修改的 Nav Area：

{% code lineNumbers="true" %}
```cpp
UNavFilter_DesertAnimal::UNavFilter_DesertAnimal() {

    // [先前代码]

    // 设置其参数
    Desert.AreaClass = UNavArea_Desert::StaticClass();

    Desert.bOverrideEnteringCost = true;
    Desert.EnteringCostOverride = 0.f;

    Desert.bOverrideTravelCost = true;
    Desert.TravelCostOverride = 0.8f;

    // [其余代码]
}
```
{% endcode %}

最后，我们需要将这个过滤区域添加到过滤区域的数组中：

{% code lineNumbers="true" %}
```cpp
UNavFilter_DesertAnimal::UNavFilter_DesertAnimal() {

    // [先前代码]

    //将其添加到过滤区域的数组中
    Areas.Add(Desert);
}
```
{% endcode %}

为了方便起见，这里是全 `.cpp` 文件：

{% code lineNumbers="true" %}
```cpp
#include "NavFilter_DesertAnimal.h"
#include "NavArea_Desert.h"

UNavFilter_DesertAnimal::UNavFilter_DesertAnimal() {

    // 创建导航过滤区域
    FNavigationFilterArea Desert = FNavigationFilterArea();

    // 设置其参数
    Desert.AreaClass = UNavArea_Desert::StaticClass();

    Desert.bOverrideEnteringCost = true;
    Desert.EnteringCostOverride = 0.f;

    Desert.bOverrideTravelCost = true;
    Desert.TravelCostOverride = 0.8f;

    //将其添加到过滤器的区域数组中
    Areas.Add(Desert);
}
```
{% endcode %}

编译这段代码，下次需要使用导航系统时就可以使用这个过滤器了。这就是关于导航过滤器的讨论内容。
