# 启用环境查询系统

EQS是一个早在Unreal 4.7中就引入的功能，并在4.9中有了很大的改进。然而，在版本4.22中，尽管EQS在许多游戏中被成功使用，但它仍被列为实验性功能，这表明EQS非常稳健。

结果，我们需要从实验功能设置中启用它。从顶部菜单中，进入编辑 | 编辑器首选项...，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
请注意不要与**项目设置**混淆。从顶部菜单中，Viewport上方，你只能访问**项目设置**。然而，从整个编辑器的顶部菜单中，你将能够找到**编辑器首选项**。前面的截图应该帮助你找到正确的菜单（即**编辑**下拉菜单）。
{% endhint %}

从侧边菜单中，你将看到一个名为“**Experimental**”的部分（在“General”类别下），如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

如果你向下滚动设置，你会找到AI类别，在这个类别中你可以启用**环境查询系统（Environment Querying System）**：

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

检查此选项旁边的框，结果是整个项目中将激活环境查询系统。现在，您将能够为其创建资源（以及扩展它），并从行为树中调用它。

{% hint style="info" %}
如果您在**AI类别**中无法看到**环境查询系统**复选框，那么您可能正在使用最新版本的引擎，其中（终于）**EQS**不再是实验性的，因此在您的项目中始终启用。如果这是您的情况，那么跳过此部分并继续进行下一部分。
{% endhint %}
