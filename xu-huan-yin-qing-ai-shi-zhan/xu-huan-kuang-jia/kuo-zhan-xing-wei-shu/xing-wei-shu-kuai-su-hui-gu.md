# 行为树快速回顾

这里是一个关于行为树的快速回顾，以供您复习。&#x20;

行为树是一种用于决策的结构，它使用黑板作为其记忆。 特别地，流程从名为根节点的特殊节点开始，一直向下到叶子节点，这些叶子节点被称为任务。任务是AI可以执行/采取的单个动作。&#x20;

然后，所有非叶子节点（或根节点）都是复合节点。复合节点选择要执行哪个子节点。两个主要的复合节点是序列（按顺序尝试执行其所有子节点的序列，如果它们成功，则报告成功，否则报告失败）和选择器（尝试每个子节点，直到找到一个成功的，报告成功，或者全部失败，报告失败）。&#x20;

复合节点和任务节点都可以使用装饰器（施加必须为真的条件，以便您可以选择该节点）或在顶部使用服务（一段持续运行的代码，例如用于设置黑板值的代码）。&#x20;

如果您还有疑问，请复习第2章，行为树和黑板。&#x20;