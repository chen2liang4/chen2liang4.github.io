---
title: 开发人员如何保证质量
---
最近招聘季面试多，公司HR和我讨论了下这个问题。我自己总结了下大概分着几个方面：

## 测试保证质量
毫无疑问，**测试是保证质量最可靠的方法**。  
测试绝不止是测试人员的事。**优秀的开发人员一定是善于测试的**，因此他们提交的代码bug少，从而在团队中内部得到认可。  
**开发人员不自测的行为是不可接受**的。在提交测试前，应充分思考下happy path，negative path和exception path进行测试，甚至可以像测试人员一样做下monkey testing。  
如果有测试团队提供的测试用例，或者进行ATDD，不但可以避免业务理解的不一致，也可覆盖到考虑不周全的地方，提升整个团队的交付效率。  

自动化测试虽然有一定的成本，但做好了的话效果也是很明显的。  
端到端的自动化测试和有些集成测试需要专人来负责，开发人员主要负责**单元测试**。  
是否进行TDD，可以由开发人员个人来决定。鉴于单元测试的成本和挑战，鼓励多做单元测试，但对覆盖率不做强制性的要求。不过重要且复杂度高的代码，是要建立单元测试任务。另外修复bug的时候，也可要求加入单元测试你进行覆盖。单元测试也可以通过结对的方式来进行，即一人写实现代码，一人写测试代码。

## 代码审查
代码审查可以保证内部质量，防止代码腐化。  
首先可以**使用工具进行代码扫描和分析**，从代码规范到性能，安全问题都可以覆盖。每个公司或项目最好都建立自己的规则集，并且保证100%通过。如果不能100%通过，在实践中就没有执行标准，到最后就无人重视成为摆设。所以建立自己的规则集是有必要的，因为现有的比较大而全，还有些比较严格，如JSLint will hurt your feelings。这种更全面和严格的规则集可以由代码审查者在人肉审查时参考使用。  

**人肉审查**可以从内部去检查是否存在测试不容易发现的缺陷，这往往需要审查者对业务功能熟悉。如果是group reivew或者是peer review，可以更多审查代码的**实现和设计中的坏味道**，如是否有重复代码，垃圾代码，可读性以及违反设计原则的地方等等。我个人就习惯通过工具找出复杂度高的代码进行审查。

## 设计保证质量
如果设计不好，好比开局选择hard模式，实现过程就困难重重，出来的质量自然没有充分保障。  
比如解决方案，技术框架或者流程，这些技术决定都会和产出质量相关。**技术决定**除了技术因素外，还需要考虑团队人员、时间等工程因素。没有好的执行，再好的技术也等于零。  

应用好的**设计思想、原则和模式**，也是保证质量的方法。如保持低偶高内聚，正确的抽象和职责划分等。  
我第一次意识到这点的时候，是发现有个API升级后相关有个功能还是不对，原因是原来的实现并没有调用期望的API，而是通过其他粒度更细的API来完成的。自然升级的API对这功能点没有任何影响。如果我们只暴露该暴露的API，则不会发生这种情况。这就是设计缺陷导致的问题。  

**过度设计**也会造成质量的下降。最近遇到的一个例子就是在web中使用了多线程，然后引起了一些莫名奇妙的问题。其实本来就没有性能问题，即使有性能问题也可考虑其他方案来解决。  

## 质量持续保证
我们不可能把每一步都做的很完美，也不可能做到真正的零缺陷发布。所以在完成开发后，依然需要对产品质量持续关注和改善。  
组织**Bug分析**可以帮助我们挖掘出一些root cause。开发过程中还留下了一些**TODO**没有完成，还有一些因为时间等关系不得不**妥协折中的实现**。团队应该对这些问题**组织重构**。  
出现了问题**能快速修复**也是质量好的表现。所以做好日志记录，对线上运行情况进行监控，有能快速重现问题的环境，都可以帮助我们分析和及时修复问题。

## 质量和成本
质量是有成本的。不计成本的保证质量是无法按期交付的。所以我们必须有所取舍，有优先级，并在内部达成一致。