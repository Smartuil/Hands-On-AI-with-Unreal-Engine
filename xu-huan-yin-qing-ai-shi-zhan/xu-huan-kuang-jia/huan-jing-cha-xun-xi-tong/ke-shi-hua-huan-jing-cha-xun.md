# 可视化环境查询

如前所述，有一个简单的内置方法可以在游戏世界中直接从视口可视化**环境查询**，甚至不需要运行游戏。实际上，有一个特殊的Pawn可以做到这一点。然而，这个Pawn不能直接导入到关卡中，因为为确保它不被误用，它在代码库中被声明为虚拟。这意味着要使用它，我们需要创建一个自己的蓝图Pawn，直接从这个特殊Pawn继承。

值得庆幸的是，在这一步之后，Pawn功能齐全，不需要更多的代码，只需与参数一起使用（即您想要可视化的环境查询）。

首先，创建一个新的蓝图。要继承的类是EQSTestingPawn，如下图所示：

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

然后，你可以将其重命名为MyEQSTestingPawn。

如果将其拖入地图，从详细信息面板中，可以更改EQS设置，如下图所示：&#x20;

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

最重要的参数是**查询模板**，您可以在其中指定要可视化的查询。如果需要深入了解该参数，请参阅第12章，AI调试方法 - 导航、EQS和性能分析。
