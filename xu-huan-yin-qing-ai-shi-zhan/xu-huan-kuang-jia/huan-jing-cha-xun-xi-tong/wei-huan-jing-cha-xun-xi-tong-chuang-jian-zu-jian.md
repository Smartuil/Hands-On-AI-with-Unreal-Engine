# 为环境查询系统创建组件

在本节中，我们将学习需要扩展哪个类来在环境查询系统中创建我们的自定义组件。

**创建上下文**

创建自定义上下文是在环境查询期间需要时获得正确引用的关键。特别是，我们将创建一个简单的上下文来检索对玩家的单一引用。&#x20;

让我们探讨如何在C++和蓝图中创建这个上下文。

**在蓝图中创建玩家上下文**

要创建一个上下文，我们需要从EnvQueryContext\_BlueprintBase类继承。在蓝图的情况下，在创建时，只需选择突出显示的类，如下图所示：

<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

关于名称，约定是保留前缀“EnvQueryContext\_”。我们可以将我们的上下文称为“EnvQueryContext\_BPPlayer”。&#x20;

对于蓝图上下文，您可以选择实现以下功能之一：&#x20;

<figure><img src="../../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

每个将为环境查询提供一个上下文。 我们可以覆盖Provide Single Actor功能，然后返回玩家Pawn，就这么简单：

<figure><img src="../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

因此，我们现在拥有了一个能够获取玩家引用的上下文。

**在C++中创建玩家上下文**

在创建C++上下文的情况下，继承自**EnvQueryContext**类，如下图所示：

<figure><img src="../../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

约定是相同的，即以"**EnvQueryContext\_**"为前缀。我们将我们的类称为"**EnvQueryContext\_Player**"：

<figure><img src="../../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

在C++中，只有一个函数需要重写：ProvideContext()。因此，我们只需要在.h文件中重写它，如下所示：

<pre class="language-cpp" data-line-numbers><code class="lang-cpp">#include "CoreMinimal.h"
#include "EnvironmentQuery/EnvQueryContext.h"
#include "EnvQueryContext_Player.generated.h"
/**
 *
 */
UCLASS()
class UNREALAIBOOK_API UEnvQueryContext_Player : public UEnvQueryContext
{
  GENERATED_BODY()
  
  virtual void ProvideContext(FEnvQueryInstance&#x26; QueryInstance,
<strong>  FEnvQueryContextData&#x26; ContextData) const override;
</strong>};
</code></pre>

在实现文件中，我们可以提供上下文。我不打算详细讲解如何操作——你可以阅读其他上下文的代码以帮助你理解这一点。无论如何，我们的.cpp文件可以有如下类似内容（我本可以采用不同的实现方式，但我选择这种方式是因为我认为它易于理解）：

{% code lineNumbers="true" %}
```cpp
#include "EnvQueryContext_Player.h"
#include "EnvironmentQuery/EnvQueryTypes.h"
#include "EnvironmentQuery/Items/EnvQueryItemType_Actor.h"
#include "Runtime/Engine/Classes/Kismet/GameplayStatics.h"
#include "Runtime/Engine/Classes/Engine/World.h"
void UEnvQueryContext_Player::ProvideContext(FEnvQueryInstance&
QueryInstance, FEnvQueryContextData& ContextData) const
{
  if (GetWorld()) {
    if (GetWorld()->GetFirstPlayerController()) {
      if (GetWorld()->GetFirstPlayerController()->GetPawn()) {
        UEnvQueryItemType_Actor::SetContextHelper(ContextData,
        GetWorld()->GetFirstPlayerController()->GetPawn());
      }
    }
  }
}
```
{% endcode %}

因此，我们能够在C++中检索玩家上下文。

**创建生成器**

与创建上下文类似，我们可以创建自定义生成器。然而，我们不会详细讨论这个，因为它们超出了本书的范围。 在蓝图的情况下，继承自EnvQueryGenerator\_BlueprintBase类，如下图所示：

<figure><img src="../../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

在C++中，你需要继承自EnvQueryGenerator：

<figure><img src="../../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
您可能想直接从EnvQueryGenerator\_ProjectedPoints开始，因为您已经准备好了所有的投影。这样做的话，您只需要关注它的生成。
{% endhint %}

**创建测试**

在当前版本的虚幻引擎中，无法在蓝图中创建测试 - 我们只能使用C++来完成。您可以通过扩展EnvQueryTest类来实现这一点：

<figure><img src="../../../.gitbook/assets/image (236).png" alt=""><figcaption></figcaption></figure>

不幸的是，这也超出了本书的范围。然而，探索虚幻引擎源代码将为您提供大量的信息和几乎无限的学习资源。
