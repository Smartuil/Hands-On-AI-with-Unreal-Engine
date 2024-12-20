# 探索内置节点

在我们创建自己的生成器、上下文和测试之前，让我们谈谈内置节点。Unreal提供了一些有用的内置通用节点。我们将在本节中探讨它们。

{% hint style="info" %}
请记住，本节将像文档一样分析解释EQS的每个内置节点的工作原理。因此，如果需要，请将此节作为参考手册，并对不感兴趣的部分跳过。
{% endhint %}

**内置上下文**

由于我们开始通过查看上下文来解释EQS，让我们从内置上下文开始。当然，制作通用上下文几乎是一个悖论，因为上下文非常特定于“情境”（情况）。

然而，Unreal提供了两个内置上下文：

* **EnvQueryContext\_Querier**：这代表了提出查询的Pawn（准确地说，不是提出查询的Pawn，而是运行行为树的Controller提出查询，这个上下文返回受控制的Pawn）。因此，通过使用这个上下文，所有内容都将相对于查询者。

{% hint style="info" %}
正如我之前提到的，在底层，查询者实际上是一个上下文。
{% endhint %}

**内置生成器**

有许多内置生成器，大多数情况下，这些生成器足以满足您想要进行的大多数环境查询（EQS）。只有当您有特定需求或想要优化EQS时，才会使用自定义生成器。这些生成器大多数都是直观的，所以我将简要解释它们，并在必要时提供截图，显示它们生成的点类型。

{% hint style="info" %}
以下截图使用了一种特殊的Pawn，它能够可视化环境查询。
{% endhint %}

这是可用内置生成器的列表，你可以在环境查询编辑器中找到它们：

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

为了组织这些信息，我将每个生成器分成一个子节，并按前一张截图中的顺序（按字母顺序）进行排序。

{% hint style="info" %}
当我提到生成器的设置时，意思是，一旦选定特定的生成器，详细信息面板中可用的选项。
{% endhint %}

**Actors Of Class**

这个生成器会获取特定类别的所有演员，并返回他们的所有位置作为生成的点（如果这些演员在上下文中的某个半径范围内）。

这是在环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

可能的选项包括**搜索的演员类别**（显然）和从**搜索中心**开始的**搜索半径**（以上下文表示）。此外，我们可以获取特定类别的所有演员，而不考虑他们是否在搜索半径内：

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

在之前的截图中，我使用了**查询器**作为**搜索中心**，**搜索半径为50000**，并将**第三人称角色**作为**被搜索的演员类**，因为该项目中已经包含了它。

通过使用这些设置（并放置了一些**第三人称角色演员**），我们得到了以下情况：

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

请注意三个**第三人称角色演员**周围的蓝色球体。

**当前位置**

当前位置生成器简单地从上下文中获取位置（或多个位置），并使用它们来生成点。

这就是它在环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

这个生成器唯一可用的设置是查询上下文：

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

因此，如果我们将**查询器**作为**查询上下文**，那么我们只有查询器自身的位置，如下面的截图所示：

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

**Composite**

**复合生成器**允许您混合多个生成器，以便拥有更广泛的点选择范围。

这就是它在环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

在设置中，您可以设置一系列生成器：

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

由于我们没有时间详细地检查所有内容，所以我不会进一步讨论这个生成器。

**要点：圆形生成器**

顾名思义，**圆形生成器**会在指定半径的圆周围生成点。此外，还提供了与导航网格交互的选项（这样你就不会在导航网格之外生成点）。

这就是它在环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

这是一个非常复杂的生成器，因此它有各种设置。让我们来看看它们：

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
理想情况下，如果每个设置都有一个截图，这样我们就能更好地了解每个设置如何影响点的生成，那就太好了。不幸的是，这本书已经有很多截图了，而且专门用一章来介绍这些复杂生成器的不同设置会占用很多时间和大量的“书本空间”。然而，有一种更好的方法可以让你获得同样的感受：亲自尝试！是的——一旦你知道如何设置EQSTestingPawn，你就可以自己尝试它们，并看到每个设置如何影响生成过程。这是你学习和真正理解所有这些设置的最好方式。
{% endhint %}

* **圆半径**： 顾名思义，它是圆的半径。
* **点间距**： 每个点之间应该有多少空间；如果圆形点间距方法设置为按间距计算。
* **点数**： 应该生成多少个点；如果圆形点间距方法设置为按点数计算。
* **圆形点间距方法**： 确定要生成的点数是否应基于固定数量的点（按点数）计算，或者基于当前圆上适合的点数（按间距）计算，如果点之间的空间是固定的。
* **弧方向**： 如果我们只生成圆的一部分弧，这个设置决定了弧的方向。计算方向的方法可以是两点（取两个上下文并计算两者之间的方向）或旋转（取一个上下文并获取其旋转，根据该旋转决定弧的方向）。
* **弧角**： 如果这个角度不同于360度，它定义了点停止生成的切割角度，从而创建一个弧而不是一个圆。这种弧的方向（或旋转）由弧方向参数控制。
* **圆心**： 顾名思义，它是圆的中心，表示为上下文。
* **生成圆时忽略任何上下文演员**： 如果选中，它将不考虑用作圆上下文的演员，从而跳过在这些位置生成点。
* **圆心Z轴偏移**： 顾名思义，它是圆心沿Z轴的偏移。
* **追踪数据**： 在生成圆时，如果有障碍物，通常我们不想在障碍物后面生成点。这个参数决定了进行“水平”追踪的规则。这些选项如下：
  * **无**：没有追踪，所有生成的点都在圆上（或弧上）。
  * **导航**：这是默认选项。NavMesh结束的地方就是点的生成位置，即使从中心到该点的距离小于半径（在某种程度上，如果遇到NavMesh的边界，圆会假设为NavMesh的形状）。
  * **几何**：与导航相同，但追踪将使用关卡的几何形状（如果你没有NavMesh，这可能非常有用）。
  * **跨越边缘的导航**：与导航相同，但现在的追踪是“跨越边缘”。
  * **投影数据**：这与追踪数据的工作原理类似，但通过从上方投影点来进行“垂直”追踪。其余的概念与追踪数据完全相同。选项有无、导航和几何，它们在追踪数据中所具有的含义相同。“跨越边缘的导航”不存在，因为它没有任何意义。

通过使用前面截图中显示的相同设置（我使用的是导航追踪数据，并且在关卡中有一个NavMesh），这就是它的样子（我用P键激活了NavMesh，所以你也可以看到）。

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

通过使用几何形状作为追踪数据，我们得到了一个非常相似但略有不同的形状：

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
如果你有一个NavMesh结束了，但关卡的几何形状并没有结束，那么效果会更加明显。
{% endhint %}

**点：圆锥体**

顾名思义，圆锥体生成器在特定的上下文（如聚光灯）中生成点。此外，还提供了与Navmesh交互的选项（这样你可以将点投影到Navmesh上）。

重要的是要理解，它的形状是由许多圆生成的，我们总是从这些圆中取出相同的弧。所以，如果我们取整个圆，我们基本上是在单个切片的区域内生成点。

{% hint style="info" %}
这个生成器也可以用来生成覆盖整个圆区域的点。
{% endhint %}

这就是它在环境查询编辑器中的样子。

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

它的设置大多与圆锥的形状有关，所以让我们一起探索它们吧：

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
再次理想的情况是，为每种设置组合提供截图，以便您了解每种设置如何影响点的生成。由于我们在这本书中没有空间这样做，我鼓励您使用EQSTestingPawn进行实验，以便您有更清晰的理解。
{% endhint %}

* 对齐点距离：这是生成点的每个弧之间的距离（即从中心角度相同的点之间的距离）。较小的值会生成更多的点，并且考虑的区域会更密集。
* 锥体度数：这决定了每个圆的弧的大小（我们考虑的是切片有多宽）。360的值考虑了整个圆的面积。
* 角度步长：这是同一弧的点之间的距离，以度数表示。较小的值意味着更多的点，考虑的区域会更密集。
* 范围：这决定了锥体可以有多远（以聚光灯为例，决定它能照亮多远）。
* 中心演员：这是生成圆的中心，用于确定锥体。它是中心，并以上下文形式表示。
* 包含上下文位置：顾名思义，如果选中，还会在锥体/圆的中心生成一个点。
* 投影数据：这通过从上方投影点来执行“垂直”轨迹，考虑几何体或导航网格。实际上，可能的选项有无、导航和几何体。

通过使用默认设置，这在关卡中可能看起来是这样的锥体：

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

**点：甜甜圈**

顾名思义，**甜甜圈生成器**在一个甜甜圈形状（或者对于数学爱好者来说，“环形”）中生成点，从一个特定的中心开始，这个中心作为一个上下文给出。此外，还有各种选项可以与导航网格进行交互（以便可以将点投影到导航网格上）。

{% hint style="info" %}
这个生成器也可以用来生成螺旋形状。就像圆锥形状一样，这个生成器可以用来生成点来覆盖整个圆的面积。你可以通过将其内半径设置为零来实现这一点。
{% endhint %}

这就是它在环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

以下是可用的设置：

<figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

* **内半径：** 这是甜甜圈“孔”的半径；在这个半径内不会生成任何点（因此，从中心到它的距离不更接近这个值）。
* **外半径**： 这是整个甜甜圈的半径；点将在内半径和外半径之间的环中生成。这也意味着在这个半径之外不会生成任何点（因此，从中心到它的距离更远）。
* **环数**：这是指在内外半径之间应生成多少环的点。这些环总是均匀间隔的，这意味着它们的距离由这个变量控制，同时受内半径和外半径的影响。
* **每环点数**：这规定每个生成环应有多少点。这些点是沿着环均匀分布的。
* **弧方向**：如果我们在生成甜甜圈的一个弧（准确地说，是生成甜甜圈的圆的弧），这个设置决定了弧的方向。计算方向的方法可以是两点（取两个上下文并计算它们之间的方向）或旋转（取一个上下文并获取其旋转，基于该旋转决定弧的方向）。&#x20;
* **弧角度**：如果这不是360度，它定义了点停止生成的切割角度，从而创建一个弧而不是一个圆。这种弧的方向（或旋转）由弧方向参数控制。&#x20;
* **使用螺旋模式** ：如果选中，每个环中的点会稍微偏移以生成螺旋图案。&#x20;
* **中心**：这是生成环的中心（以及用内半径和外半径指定的甜甜圈的最小和最大扩展部分）。它表示为一个上下文。&#x20;
* **投影数据**：这通过从上方投影点来执行“垂直”轨迹，考虑几何体或导航网格。可能的选项有None、Navigation和Geometry。

为了理解这些设置，请看以下截图：

<figure><img src="../../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

通过使用这些稍作修改的设置（请注意我是如何增加内半径、增加环数和每环点数，以及为投影数据使用导航的），可以轻松可视化甜甜圈。以下是我使用的设置：\


<figure><img src="../../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

这是他们产生的结果：

<figure><img src="../../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

通过使用相同的设置，并勾选“使用螺旋模式”，可以看到不同环中的点稍微偏移，从而形成螺旋图案：

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

**点：网格**

如其名，网格生成器在一个网格内生成点。此外，还有选项可以与导航网格互动（这样你就不会在导航网格之外生成点）。

这是环境查询编辑器中的样子：

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

这个生成器的设置相当直接明了：

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

* **GridHalfSize**: 网格应该从其中心延伸多少（这意味着它是完整网格大小的一半）。网格的尺寸完全由这个参数以及Space Between决定。
* **Space Between**: 网格的每一行和每一列之间有多少空间。网格的尺寸完全由这个参数以及GridHalfSize决定。
* **Generate Around**: 这是网格的中心（开始生成的地方），并以上下文的形式表达。
* **Projection Data**: 这通过从上方投影点来执行“垂直”追踪。它通过考虑几何体或导航网格来实现这一点。可能的选项有None、Navigation和Geometry。

通过查看设置，可以看出这个生成器相当简单，但却强大且非常常用。使用默认设置，这在关卡中的样子如下（在Navmesh上启用投影，并存在于地图中）：

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

**点：路径网格**

顾名思义，**路径网格生成器**就像网格生成器一样，在一个网格内生成点。然而，这个生成器的不同之处在于，它会检查这些点是否在指定距离内，通过生成设置中指定的上下文（通常是查询者）可以到达。 在环境查询编辑器中是这样的：

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

这个生成器的设置与点阵：网格生成器几乎完全相同：

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

* **路径到物品**：如果勾选，这将排除所有从上下文中无法到达的点，在查询者的设置中。&#x20;
* **导航过滤器**：顾名思义，它是用来执行寻路的导航过滤器。
* **网格半尺寸**：这表示网格应从其中心向外延伸多少（这意味着它是完整网格大小的一半）。网格的尺寸完全由这个参数以及间距决定。&#x20;
* **间距**：这表示网格的每一行和每一列之间的空间有多少。网格的尺寸完全由这个参数以及网格半尺寸决定。&#x20;
* **生成围绕**：这是网格的中心（开始生成的地方），并以上下文的形式表示。&#x20;
* **投影数据**：通过从上方投影点来执行“垂直”追踪。这是通过考虑几何体或导航网格来实现的。可能的选项是**无**、**导航**和**几何体**。&#x20;

这就是它在环境中的样子（我稍微改变了关卡以阻挡楼上的路径。这使得楼梯后的那些无法到达的点甚至没有被这个生成器生成变得很明显）：

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

**内置测试**

现在我们已经探讨了所有的生成器，是时候探索引擎内可用的不同测试了。通常，返回值可以是布尔值或浮点数。

返回浮点数的测试通常用于评分，而返回布尔值的测试则更常用于过滤。然而，根据测试是用于过滤还是评分，每个测试可能有不同的返回值。&#x20;

以下是可能的内置测试列表；让我们来探讨一下：&#x20;

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

* 距离：计算项目（生成的点）与特定上下文（例如查询者）之间的距离。它可以在3D、2D、沿z轴或沿z轴（绝对）计算。返回值是浮点数。&#x20;
* 点积：计算线A和线B之间的点积。两条线都可以表示为两个上下文之间的线，或者表示为特定上下文的旋转（通过取旋转的前进方向）。计算可以在3D或2D中完成。&#x20;
* 游戏标签：对游戏标签执行查询。&#x20;
* 重叠：使用盒子执行重叠测试；可以指定一些选项，如偏移量或扩展，或重叠通道。
* 寻路：在生成的点与上下文之间执行寻路。我们可以指定返回值是布尔值（路径是否存在）还是浮点数（路径成本或路径长度）。此外，可以指定路径是从上下文到点还是相反方向，并且可以使用导航过滤器。&#x20;
* 批量寻路：与寻路相同，但是以批处理方式。
* 投影：执行投影，通过不同的参数进行自定义。&#x20;
* 追踪：执行追踪测试，具有引擎其他地方执行追踪的所有可能选项。这意味着它可以追踪线、盒子、球体或胶囊；在可见性或相机追踪通道上；复杂或简单；从上下文到点，或相反方向。

&#x20;这就结束了我们对内置节点的探索。
