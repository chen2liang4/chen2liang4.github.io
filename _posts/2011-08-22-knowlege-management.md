---
title: 碎片知识管理
---

为项目增加了一个VS工程，因为daily build环境已经搭好了，并且已运行了一段时间。所以当我把新工程提交到SVN后，项目应该如往常一样自动发布。但事实并非如此，我折腾了很久，第二天在别人的帮助下才解决了问题。下面是遇到的主要问题：
1. 当我为了确定部署服务器获取了正确代码，试图手动编译时遇到了目录拒绝访问错误，我不得以把相关目录的安全性降低为everyone full control。实际上，我需要以Administrator权限运行Visual Studio，服务器是Windows 2008 server。
2. 手动编译成功后，仍然自动发布失败。查了半天CC.net的日志，也未找到问题所在。第二天才知道是发布目录不存在导致的失败。
3. 即使发布目录存在，仍然不能成功。因为在daily build环境中是以Dev, Test两个编译选项编译的，而新工程没有增加这两个编译选项。

解决方法很简单，但是知道答案的过程却很曲折。如何使用CC.net的文档写了，对CC.net也不算陌生，但如果不是同事亲历过这些问题，并暂时没有忘记原因的话，我还要花更多的时间。换句话说，I wear my colleague's shoes。

同样在上周发生的，关于如何修改一个系统中他人的联系信息。这个的年代就稍微久远点，team中接触这个系统最久的同学也想不起来需要什么样的权限才能修改他人的联系信息。数据库中的权限表描述了每个权限的范围，没有一个看起来和联系信息相关。还好我们有一个可运行的开发测试环境，可以赋予不同的权限来试，但试完了也没结果。最后全靠记忆闪了下光，果然，这个功能是在源码中写死了给Administrator的。

这些知识信息就如碎片一样，整合到文档中，不好描述而且会使文档很臃肿。而臃肿的文档是不适合维护的，最终会导致文档描述与事实不符。大多时候还是靠口口相传，前赴后继的反复摔同样的跟斗，把这些碎片知识传递下去。但是这种传递方式实在是太折磨人了，常常能得到WTF抱怨，这是什么破系统！！！

回头看看参与者众多的开源系统和产品，文档不多，能出较详细帮助的，基本算是很受欢迎的“大”项目了。除了系统本身做的好，不需要太多文档化工作外，不能忽视社区的力量。大量的问题会在论坛中被解答，而通过搜索引擎，很多问题又都能在论坛中找到答案。热心观众回答问题，就如我们项目中的口口相传，老员工告诉新人答案。这种方式，随着时间的久远以及人员的变更，会越发的低效。而在一般公司项目中，搜索答案更是一向比较缺乏。就这点来说，产品论坛，FAQ这样的形式还是比较易于传递碎片信息。

所以，我个人对于这些碎片知识，觉得下面这些方法来处理或许会有点用：
1. 智能化系统  
系统相对智能些，能应对复杂的应用场景，自动处理。如当目录不存在时，自动创建目录，自然就避免了错误的发生，也不用再管理发布前必须要确保目录的存在这样的知识。
2. 可维护系统  
提供详尽，准确的诊断信息，对错误排查提供有效帮助。很遗憾cc.net这点做的不是很好。
3. 自我解释系统  
对于用户来说，在设计上让人一看就知道咋用。次之，在应用中提供帮助，如一些tip，如网站中的site map；再如Enwisen配置工具，本身就带了大量操作说明。 
对于源代码，代码应做到易懂，函数名，变量名已经说明了问题。甚至提倡，对一段不好描述的代码抽象成函数，通过函数名来解释。次之，加上简洁的comments。如在数据库权限表中，就增加了description字段对该权限进行解释。
4. 提供FAQ, Wiki和Discussions  
这种形式不像目录化的文档，形式比较散，比较好维护碎片信息，又不会破化文档的目录化。通过搜索引擎查答案也是一种最乐于接受的方式。SharePoint在这方面也提供了便捷。本来嘛，SharePoint就是知识管理工具。