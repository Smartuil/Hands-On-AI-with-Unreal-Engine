# 安装和准备软件

在您继续阅读之前，让我们安装我们需要的软件。特别是，我们需要Unreal Engine和Visual Studio。

**Unreal Engine**

让我们来谈谈Unreal Engine。毕竟，这是一本关于如何在令人惊叹的游戏引擎中开发游戏AI的书。

Unreal Engine是一款由Epic Games开发的游戏引擎，首次发布于1998年。如今，它是使用最广泛的（开源）游戏引擎之一，这要归功于其强大的功能。以下截图展示了Unreal Engine的主界面：

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

我们需要安装最新版本的Unreal Engine。您可以通过访问以下网址找到它：https://www.unrealengine.com/en-US/what-is-unreal-engine-4。除非您从源代码（https://docs.unrealengine.com/en-us/Programming/Development/BuildingUnrealEngine）获取Unreal Engine，否则您将需要安装Epic Launcher。如果您是蓝图用户，并且不打算使用C++，那么这些步骤就足够了。另一方面，如果您要使用C++，那么您还需要执行一些额外的步骤。

在安装引擎时，您需要检查一些选项（如果您使用的是C++）。特别是，我们需要确保同时拥有“**引擎源**”和“**调试符号**”的选项，如下面的屏幕截图所示：

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption><p><strong>Unreal Engine 4.22.0 安装选项</strong></p></figcaption></figure>

通过这样做，我们将能够浏览C++引擎源代码，并在发生崩溃时拥有一整套调用记录（以便您知道哪里出了问题）。

**Visual Studio**

如果您使用的是Blueprint（仅适用于C++用户），那么您不需要这个IDE来编辑我们的C++代码。我们将使用Visual Studio，因为它与Unreal引擎完美集成。您将能够从官方网站免费下载Visual Studio Community Edition：https://www.visualstudio.com/ 或从 https://visualstudio.microsoft.com/ 下载。

您还可以找到这个简短的指南，说明如何设置Visual Studio，使其与Unreal Engine一起使用：https://docs.unrealengine.com/en-us/Programming/Development/VisualStudioSetup

一旦你已经安装了所有需要的东西并准备好开始，我们可以继续阅读本章的内容。

{% hint style="info" %}
如果你是 macOS 用户，有适用于 macOS 的 Visual Studio 版本。你也可以使用 XCode。
{% endhint %}
