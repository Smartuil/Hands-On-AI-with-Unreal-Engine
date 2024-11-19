# 不同团队

在Unreal内置的AI感知系统中，AI和任何可以被检测的东西都可以有一个团队。有些团队相互对立，而有些则只是中立。因此，当AI感知到某物时，该物体可以是友好的（它在同一个团队中）、中立的或敌对的。例如，如果AI正在巡逻一个营地，我们可以忽略友好和中立的实体，只关注敌人。顺便说一下，默认设置是只感知敌人。&#x20;

您可以通过Sense的Detecting for Affiliation设置来更改AI可以感知的实体类型。

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

这提供了三个复选框，我们可以选择希望AI感知的内容。&#x20;

总共有255个团队，默认情况下，每个实体都在团队255中（唯一的特殊团队）。无论实体是否在同一个团队中，只要在团队255中的实体都被视为中立。否则，如果两个实体在同一个团队（不同于255），它们会互相视为友好。另一方面，两个实体在不同团队（不同于255）中，它们会互相视为敌人。&#x20;

现在的问题是，我们如何改变团队？目前，这只能在C++中实现。此外，我们谈论了实体，但谁实际上可以成为团队的一员？ 实现**IGenericTeamAgentInterface**的所有内容都可以成为团队的一部分。 AIControllers已经实现了它。因此，在AI控制器上更改团队很容易，如下面的代码片段所示：&#x20;

{% code lineNumbers="true" %}
```cpp
// Assign to Team 1
SetGenericTeamId(FGenericTeamId(1));
```
{% endcode %}

对于其他实体，一旦它们实现了**IGenericTeamAgentInterface**，它们可以覆盖GetGenericTeamId()函数，这为AI提供了一种检查该实体所属团队的方法。
