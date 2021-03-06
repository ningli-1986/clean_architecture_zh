# 第八章 开闭原则（OCP）

![](/assets/8/c8.png)

开闭原则由Bertrand Meyer在1988年发明，原则描述为：

> 软件工件应当对扩展开放，对修改关闭

换句话说，软件工件的行为应当是不必修改工件而可扩展的。

当然这也是我们研究软件架构的最基础的原因。显然，如果对需求的简单扩展迫使软件发生巨大的变化，那么该软件系统的架构师就已经犯下了一个惊人的错误。

大多数学习软件设计的学生认识到OCP作为一个原则引导着他们类和模块的设计。但是，但我们考虑架构组件层面，这个原则有着更大的意义。

用个例子会更明白。

## 想法实验

假设此时我们有个财务汇总的web页面的系统。上面的数据是滚动显示的，并且负数将显示成红色。

现在假设利益相关者想把这些同样的信息在黑白打印机打印成报告，这个报告有格式要求，需要有正确的页头，页脚和列标签，负数用括号标注。

很明显，得写新代码了，但老代码得变多少呢？

一个好的软件架构将会把改变得代码总量尽可能降到最低，理论上，一行不变。

怎么做？通过把会由于不同原因变化得事情分割开来（单一职责原则SRP），恰当地组织这些事情得依赖（依赖倒置原则DIP）。

应用SRP，我们可能提出这样得数据流视图如图8.1。一些analyser考察财物数据和产生需要报告的数据，将其交由不同的reporter处理。![](/assets/8/Figure_8.1_Applying_the_SRP.png)图8.1 应用SRP

这个想法的本质是生成报告设计到两个不同的职责：计算报告数据，数据的web或打印表格的呈现。

做了分离之后，我们需要组织源码的依赖以确保这之中职责的变化不会导致其它组件，并且新的组织确保可以不修改的做扩展。

我们把处理分成了多个类，把这些类分成了多个组件，如图8.2中双线的图表，左上方的是controller，右上方是Interactor，右下方是是Database，最后，左下方的四个组件代表Presenters和Views。

用&lt;I&gt;标记的类表示接口，标记&lt;DS&gt;表示数据结构，人角箭头表示调用关系，三角箭头表示实现或继承关系。

![](/assets/8/Figure_8.2_Partitioning_the_processes_into_classes_and_separating_the_classes_into_components.png)

图8.2 把处理分成了多个类，把这些类分成了多个组件

第一件事应该去关注所有的依赖都是源码依赖，从类A执行类B的箭头表示源码里类A用到了类B的名字，但类B没有用到类A的名字。因此，图8.2，FinancialDataMapper类由于实现关系，晓得FinancialDataGateway类，但FinancialDataGateway类并不晓得FinancialDataMapper类。

下一件事应该关注每个双线框之间的关系都是单项指向的。这意味着所有组件的关系都是单向的，组件关系如图8.3，这些箭头指向了我们要避免被变化影响的组件。

![](/assets/8/Figure_8.3_The_component_relationships_are_unidirectional.png)

图8.3 各组件间关系是单向的

让我再说一次，如果组件A应该避免被组件B的变化影响，那么组件B英爱依赖组件A。

我们想要Controller避免被Presenters的变化影响，我们想要Presenters避免被Views的变化影响，我们想要Interactor避免被任何组件影响。

为什么Interactor有如此特殊的地位，因为它包含了业务规则。Interactor包含了应用里最高层的策略。所有其他组件解决外围的问题，而Interactor解决核心问题。

虽然Controller对于Interactor是外围组件，但它对于Presenters和Views是核心组件。而且Presenters对于Controller是外围组件，但对于Views，它是的核心组件。

注意这里是怎样基于“层”创建避免被修改影响的层次关系。Interactors是最高层的概念，他们是最能避免影响的，Views是最底层的概念，所以他们呢是最容易被影响的，Presenters比Views层次要高，但低于Controller和Interactor。

## 方向控制

如果看了前面的类设计觉得难以理解，请再看一遍。范式里的多复杂度是为了确保组件之间的依赖的指向正确。

比如，FinancialDataGateway接口，在FinancialReportGenerator类和FinancialDataMapper类之间，存在倒置依赖，不然将会从Interactor组件指向Database组件。同样地，FinancialReportPresenter接，两个Views接口，他们各自所处的位置也存在依赖倒置。

## 信息隐藏

FinancialReportRequester接口有不同的目的。它避免FinancialReportController知晓Interactor组件的内部信息。如果没有这个接口，Controller将会传递性依赖于FinancialEntities。

传递性依赖违反了一般原则：软件实体不应该依赖于他们不之间用的东西。之后我们谈到接口隔离原则和共同重用原则我们还会讲到这条一般原则。

所以，即使我们更高优先级保护Interactor避免被Controller变化影响，我们也隐藏了Interactor的内部信息，避免Interactor的改变影响Controller。

## 小结

开闭原则是系统架构隐藏的驱动力量，它的目标是让系统不发生重大的修改而易于扩展。这个目标是通过分割系统成各个组件，规划组件的依赖岑寂关系，保护高层级避免受低层级组件影响实现的。

