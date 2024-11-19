# Detour Crowd

另一个内置的虚幻系统是Detour Crowd。它基于Recats库，与RVO不同，它会考虑代理移动的Navmesh。这个系统已经基本可以直接使用，但让我们深入了解它是如何工作的，以及我们如何使用它。

**Detour Crowd系统的工作原理**

在你的游戏世界中存在一个名为DetourCrowdManager的对象。它负责协调游戏中的群体。特别是，注册到DetourCrowdManager的代理将被考虑在内。DetourCrowdManager接受实现ICrowdAgentInterface的任何东西，该接口向管理器提供有关代理的数据。

{% hint style="info" %}
实际上，在底层，Detour Crowd Manager使用的是由Mikko Mononen在Recast库中开发的Detour Crowd算法，该算法已经被Epic Games为满足他们的需求而稍作修改。因此，Detour Crowd组件提供了虚幻框架和Recast Detour之间的接口。你可以通过查看本节末尾的资源来了解更多关于这方面的信息。
{% endhint %}

您可以实现ICrowdAgentInterface来创建一个代理。然而，虚幻引擎为您提供了一个名为UCrowdFollowingComponent的特殊组件，它实现了ICrowdAgentInterface以及其他功能。因此，任何具有UCrowdFollowingComponent的对象都有资格成为人群管理器中的代理。实际上，该组件会自动将自己注册到人群管理器，并激活绕行行为。

为了简化操作，ADetourCrowdAIController是一个预先制作好的控制器，它会自动将UCrowdFollowingComponent添加到控制器本身。因此，这次是由AI控制器直接从角色移动组件触发系统。

下图有助于解释这一点：

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

**使用绕行人群系统**

使用绕行人群系统最简单的方法是让你的AI控制器继承自绕行人群AI控制器，如下图所示：&#x20;

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

在C++中，你需要继承自ADetourCrowdAIController（或将UDetourCrowdFollowingComponent添加到你的控制器）。

一旦为所有想要使用Detour Crowd的控制器完成此操作，系统将基本上即插即用。&#x20;

**Detour Crowd设置**

如果您使用UCrowdFollowingComponent，该组件将通过使用角色移动组件中的避让设置（如果有的话）来实现ICrowdAgentInterface。

&#x20;因此，我们在RVO部分看到的所有避让设置都将被Detour Crowd考虑在内。因此，下面截图中突出显示的所有设置对于我们的AI角色仍然有效：&#x20;

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
请注意，由于避让权重是RVO特定参数，因此Detour系统不会考虑它。
{% endhint %}

&#x20;因此，我们看到的所有蓝图功能（例如更改组掩码）也都是有效的。

这些是每个角色特定的设置，但可以调整整体的Detour Crowd设置。要这样做，请导航到项目设置，在引擎部分下，您会找到一个名为Crowd Manager的条目，如下图所示：&#x20;

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

从这里，我们可以访问所有的Detour Crowd Manager设置。这些设置大多来自原始的Recast Crowd算法，Unreal Detour Crowd Manager提供了一个界面，您可以在其中设置算法中的这些变量。让我们从较简单的开始：&#x20;

* 最大代理数量：这是Crowd Manager将处理的最大代理数量。当然，数字越大，您可以一次性放置的代理就越多，但这会影响性能。您应该仔细计划游戏需要多少代理。此外，如果您查看源代码，这个数字将用于分配Crowd Manager处理代理所需的内存。在内存较低的情况下，记住这一点很有用。
* 最大代理半径：这是从人群管理器绕行的代理可以移动的最大范围。
* 最大避开代理数：这是绕行系统考虑的最大代理数量，也称为邻居。换句话说，这指定了应该考虑多少邻居代理（最大值）来避免碰撞。
* 最大避开墙壁数：这是绕行系统应该考虑的最大墙壁数量（通常为障碍物段）。它的作用类似于最大避开代理数，但是询问需要考虑系统周围障碍物段的数量。
* 导航网格检查间隔：这是为了实施一个离开导航网格的代理应该每隔多少秒检查和重新计算其位置（系统会尝试将代理推回导航网格上）。
* 路径优化间隔：这以秒为单位检查代理应该多久尝试重新优化自己的路径。
* 分离方向夹紧：当另一个代理在后面时，该值指示左右夹紧的分离力（前进方向和dirToNei之间的点积；因此，值-1表示禁用此分离行为）。
* 路径偏移半径乘数：当代理靠近拐角转弯时，会对路径应用一个偏移。这个变量是此偏移的乘数（这样您可以自由减少和增加此偏移）。
* 解决碰撞：尽管绕行系统做出了最大努力，代理可能仍然会发生碰撞。在这种情况下，应该由绕行系统处理碰撞（此变量的值为true）。在这种情况下，此变量设置为false，代理将使用角色移动组件。该组件将负责解决碰撞。&#x20;

如下截图所示的避免配置参数是绕行人群算法内采样方式的核心：

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

回避配置是一个包含不同采样配置的数组，参数设置略有不同。默认情况下，有四个配置，分别对应不同的采样回避质量：低、中、好和高。

质量等级是在UCrowdFollowingComponent中通过AvoidanceQuality变量设置的，该变量使用ECrowdAvoidanceQuality枚举。如果你有对UCrowdFollowingComponent的引用，你可以使用SetCrowdAvoidanceQuality()函数。

回到设置，如果你想添加或删除一个配置，你需要创建自己的UCrowdFollowingComponent版本（或者，你可以继承它并覆盖函数），这需要考虑不同数量的配置。

{% hint style="info" %}
然而，更改配置数量意味着你的游戏/应用程序正在特别使用Detour系统！
{% endhint %}

不改变配置数量的情况下，可以更改这四个质量配置的设置。以下截图显示了这些参数（来自第一个配置）：&#x20;

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

要完全理解这些设置，你应该了解算法的工作原理，但让我们尝试在不了解的情况下理解它。&#x20;

算法中进行采样的部分首先在中心点周围创建一组环（数量在自适应环参数中指示），中心点是代理最初的位置，由于速度偏置参数，会偏向速度方向。每个环都通过自适应划分进行采样（划分）。然后，算法通过使用一组较小的环递归地细化搜索，这些环以先前迭代中最佳样本为中心。算法重复此过程自适应深度次。在每次迭代中，通过考虑以下因素选择最佳样本，不同参数决定了权重（与其他因素相比，考虑的重要性）：&#x20;

* 代理的方向是否与当前速度匹配？权重是期望速度权重。
* 代理是否向侧面移动？权重是侧面偏置权重。
* 代理是否与任何已知障碍物相撞？权重为 ImpactTimeWeight（如果代理在ImpactTimeRange秒内使用当前速度相撞，它会考虑代理的当前速度扫描一个范围）。

下图应有助于您了解不同的参数：

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

调试绕行人群管理器 人群管理器与视觉记录器集成在一起，这意味着，通过一些工作，我们可以直观地调试绕行人群管理器。我们将在第13章“游戏调试器”中更详细地探讨这一点，在该章节中，我们将进一步了解视觉记录器。

**更多人群资源**

如果你想扩展对Detour Crowd算法的了解，或者探索其他替代方案，这里有一些资源：

* Mikko Mononen的原始Recast库：https:/​/​github.​com/ recastnavigation/​recastnavigation
* 一个包含许多有趣的研究算法的集合，用于处理人群：http:/​/​gamma.​cs.​unc.​edu/​research/​crowds

当然，也欢迎你自己继续探索！&#x20;
