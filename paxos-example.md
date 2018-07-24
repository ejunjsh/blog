Paxos示例

这篇文章通过一个有效的例子描述了一个名为Paxos 的分布式一致性算法。

分布式一致性算法用于使一组计算机能够就单个值达成一致，例如通常使用两阶段或三阶段提交做出的提交或回滚决策。只要选择一个值，算法的其他值就没有关系了。

在分布式系统中，这很难，因为机器之间的消息可能会丢失或无限期延迟，或者机器本身可能会发生故障。

Paxos保证节点只会选择单个值（意味着它保证安全），但不保证在大多数节点不可用时能不能去到值

# 一般的做法

一个Paxos的节点可以采取任何或所有三个角色：`proposer`，`acceptor`和`learner`。

一个`proposer`提议一个值是需要同意才行的，它发一个包含值的提议给所有的`acceptor`，`acceptor`决定是否同意这个值。

每个`acceptor`独立选择一个值--它可能收到多个来自不同`proposer`的提议--并将其决定发送给`learner`，以确定是否已接受任何值。

对于Paxos接受的值，大多数`acceptor`必须选择相同的值。实际上，单个节点可以承担许多或所有这些角色，但在本节的示例中，每个角色都在一个单独的节点上运行，如下所示。

[![](https://angus.nyc/wp-content/uploads/2012/06/0.png)](https://angus.nyc/wp-content/uploads/2012/06/0.png)
图1：基本Paxos架构。一些`proposer`向`acceptor`提出建议。当`acceptor`接受一个值时，它会将结果发送给`learner`节点。

# Paxos示例

在标准的Paxos算法中，`proposer`向`acceptor`发送两种类型的消息：**准备**和**接受**请求。

在该算法的第一阶段，`proposer`向每个`acceptor`发送包含建议值v和提议号n的准备请求。

对于其他`proposer`的提案号，每个`proposer`的提议号必须是正数的，单调递增的，唯一的，自然的数字。

在下面说明的示例中，有两个`proposer`，两个都提出准备请求。来自`proposer A`的请求和来自`proposer B`的请求首先到达`acceptor X`和`acceptor Y`，而来自`proposer B`的请求首先到达`acceptor Z`。

[![](https://angus.nyc/wp-content/uploads/2012/06/2.png)](https://angus.nyc/wp-content/uploads/2012/06/2.png)
图2：proposer A和B各自向每个接受者发送准备请求。在这个例子中，proposer A的请求首先到达接acceptor X和Y，而proposer B的请求首先到达acceptor Z.

如果接收准备请求的`acceptor`没有看到另一个提议，则`acceptor`以准备响应作出响应，该准备响应承诺永远不接受具有较低提议编号的另一提议。

这在下面的图3中说明，其显示了每个接受者对他们收到的第一个准备请求的响应。

[![](https://angus.nyc/wp-content/uploads/2012/06/3.png)](https://angus.nyc/wp-content/uploads/2012/06/3.png)
图3：每个`acceptor`响应它收到的第一个准备请求消息。

最终，`acceptor Z`接收`proposer A`的请求，`acceptor X`和`acceptor Y`接收`proposer B`的请求。

如果`acceptor`已经看到具有更高提议号的请求，则忽略准备请求，`proposer A`对`acceptor Z`的请求就是这种情况。

如果`acceptor`没有看到更高编号的请求，它再次承诺忽略具有较低提议编号的任何请求，并发回其已接受的编号最高的提议以及该提议的值。

`proposer B`对`acceptor X`和`acceptor Y`的请求就是这种情况，如下图所示：

[![](https://angus.nyc/wp-content/uploads/2012/06/4.png)](https://angus.nyc/wp-content/uploads/2012/06/4.png)
图4：acceptor Z忽略了proposer A的请求，因为它已经看到了更高编号的提议（4> 2）。acceptor X和Y用他们先前确认的最高请求来响应proposer B的请求，并承诺忽略任何编号较低的提议。

一旦`proposer`收到大多数`acceptor`的准备响应，它就可以发出接受请求。

由于`proposer A`仅收到表明没有先前提案`[no previous]`的响应，因此它向`acceptor`发送与初始提案相同的提议编号和值的接受请求（n = 2，v = 8）。

然而，这些请求被每一个`acceptor`忽略，因为`acceptor`都承诺不接受的提议号低于请求4（响应准备请求给`proposer B`）。

`proposer B`向每个`acceptor`发送包含其先前使用的提议号（n = 4）的接受请求，并且这个接受请求还包含了在其收到的准备响应消息中与最高提议号相关联的值（v = 8）。

请注意，这不是`proposer B`最初提出的值，而是它看到的准备响应消息中的最高值。

[![](https://angus.nyc/wp-content/uploads/2012/06/5.png)](https://angus.nyc/wp-content/uploads/2012/06/5.png)
图5，`proposer B`发送一个接受请求给每个`acceptor`,这个请求包含了它之前的提议号(4)和它从[n=2,v=8]中看到的值（8）

如果`acceptor`收到的接受请求的提议号大于或等于之前它保证的，那么它接受并向每个`learner`节点发送通知。

当`learner`发现大多数`acceptor已接受某个值时，Paxos算法会选择这个值，如下所示：

[![](https://angus.nyc/wp-content/uploads/2012/06/6.png)](https://angus.nyc/wp-content/uploads/2012/06/6.png)

一旦Paxos选择了一个值，与其他`proposer`的进一步沟通就无法改变这个值。

如果另一个`proposer`（`proposer C`）发送的提议号比之前看到的提议号更高，并且具有不同的值（例如，n = 6，v = 7），则每个`acceptor`都会使用之前的最高提议号进行响应（n = 4，v = 8）。

这要求`proposer C`发送包含[n = 6，v = 8] 的接受请求，该请求仅确认已经选择的值。此外，如果一些少`acceptor`还没有选择一个价值，这个过程可以确保他们最终就同一价值达成共识。

Lamport和Baker等人在论文中讨论了对标准Paxos算法的各种效率改进。例如，如果`proposer`知道它是第一个建议值的话，则准备请求不再是必须的了。

因为这种请求的提议编号为0，如果收到任何更高编号的请求的话，这种请求就会被忽略的。


翻译 https://angus.nyc/2012/paxos-by-example/
