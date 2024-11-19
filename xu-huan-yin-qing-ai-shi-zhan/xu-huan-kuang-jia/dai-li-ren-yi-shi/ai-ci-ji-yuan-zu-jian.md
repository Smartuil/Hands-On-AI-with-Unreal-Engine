# AI刺激源组件

我们已经看到了AI如何通过感官感知，但刺激最初是如何产生的呢？&#x20;

**所有的棋子都会被自动检测**

在视觉感知的情况下，默认情况下，所有的棋子已经是刺激源。实际上，在本章后面，我们将使用玩家角色，AI将会检测到他，而不需要**AIStimuliSourceComponent**。如果你有兴趣禁用这个默认行为，你可以进入你的项目目录，然后进入**Config**文件夹。在那里，你会找到一个名为**DefaultGame.ini**的文件，在该文件中你可以设置一系列配置变量。如果你在文件末尾添加以下两行，棋子默认将不会产生视觉刺激，它们将需要**AIStimuliSourceComponent**以及其他一切：

{% code lineNumbers="true" %}
```ini
[/Script/AIModule.AISense_Sight]
bAutoRegisterAllPawnsAsSources=false
```
{% endcode %}

在我们的项目中，我们不打算添加这些行，因为我们希望在没有添加更多组件的情况下检测到棋子。 **AIStimuliSourceComponent 在蓝图中**

像其他组件一样，它可以被添加到蓝图中：

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

如果您选择它，您将在详细信息面板中看到它只有两个参数：&#x20;

* **自动注册为来源**：顾名思义，如果选中，来源将自动在感知系统内注册，并且它将从一开始就提供刺激。
* **为感官注册为来源**：这是该组件提供刺激的所有感官的数组。

&#x20;关于这个组件没有太多要说的了。它非常简单易用，但很重要（您的AI可能无法感知任何刺激！）。因此，请记住在您希望它们生成刺激（可以简单地被AI看到）时将其添加到非Pawn实体中。&#x20;

**C++中的AIStimuliSourceComponent**

在C++中使用此组件很简单，只需创建它，配置它，它就可以使用了。 这是您需要使用的#include语句，以便您可以访问组件的类：

{% code lineNumbers="true" %}
```cpp
#include "Perception/AIPerceptionStimuliSourceComponent.h"
```
{% endcode %}

像其他组件一样，您需要在.h文件中添加一个变量来跟踪它：

{% code lineNumbers="true" %}
```cpp
UAIPerceptionStimuliSourceComponent* PerceptionStimuliSourceComponent;
```
{% endcode %}

接下来，你需要在构造函数中使用CreateDefaultSubobject()函数来生成它。

{% code overflow="wrap" lineNumbers="true" %}
```cpp
PerceptionStimuliSourceComponent =
CreateDefaultSubobject<UAIPerceptionStimuliSourceComponent>(TEXT("PerceptionStimuliComponent"));
```
{% endcode %}

然后，您需要按照以下方式注册一个感知源（在这个例子中是视觉，但您可以将TSubClassOf()更改为您需要的感知）：

{% code lineNumbers="true" %}
```cpp
PerceptionStimuliSourceComponent->RegisterForSense(TSubclassOf<UAISense_Sight>());
```
{% endcode %}

{% hint style="info" %}
**自动注册为来源**的布尔值默认受保护且为真。
{% endhint %}
