# 使用rails6 开发纯后端 API 项目


## 前言

本书是结合作者实战经验和心得，参考了国内外诸多大神的资料，本着一颗学习、总结与分享的心书写而成，希望对读者有着有益的指导作用，如果能指导读者并能帮助读者创造一点点价值，那我就会更加开心了！

### 本书简介

书中实现了一个简易商城系统的 纯后台 API 项目！适用于目前流行的使**用http或https协议**通信并使用**json作为数据交互**的**restful风格**的**前后端分离**项目。使用的技术栈包括 L N M R R【linux nginx mysql redis ruby 】 等！

本书分为13大章节

- 第一章：环境准备工作
- 第二章：API 版本的管理
- 第三章：实战项目分析
- 第四章：用户模块的开发
  - 模型相关开发
  - 控制器相关开发
- 第五章：用户状态管理模块开发
- 第六章：商铺模块开发
  - 模型相关开发
  - 控制器相关开发
- 第七章：分页和返回值定制
- 第八章：商品模块开发
  - 模型相关开发
  - 控制器相关开发
- 第九章：订单模块开发
  - 模型相关开发
  - 控制器相关开发
- 第十章：订单与商品模块关系开发
  - 模型相关开发
  - 控制器相关开发
- 第十一章：异常处理
- 第十二章：项目优化
- 第十三章：项目部署



前两章是开发的准备工作，从第三章开始进入项目开发阶段，按照用户、商铺、商品、订单的顺序进行开发，中间还穿插了一些重要的知识点，比如分页和返回值格式的定制！第十一和十二两章是对项目的优化，分别从异常的处理和项目本身的优化来研讨，最后一章是项目的线上部署，我们将采用 nginx+puma+mina 实现项目的自动部署。

每一章节的知识都很丰富，也是我对自我知识的总结和分享，希望大家能够从中有所收获！



### 你将收获

- 使用 Git 进行版本控制；
- 在Rails中使用 JWT 进行用户状态验证；
- 定制返回的 json 数据；
- 单元测试的编写并使用 TDD 推进开发；
- 跨域问题的解决；
- 使用 nginx + ssl + puma + mina 进行项目部署；
- 邮件服务的配置；
- Rails 中 redis缓存的使用；
- Rails中的一些编程技巧；



### 本书约定

为了更好的阅读及理解文中的知识点，我对每一章甚至是每一个小的模块都做了相对统一的结构，从浅入深，从知识点的梳理到应用的开发，所以，每一个模块的开发基本上都遵循了下面的结构：

- **基础知识储备**
  - 当前模块用到的ruby和Rails中知识点的补充和说明
- **版本控制**
  - 生成分支
- **模块开发**
  - 功能分析
  - 测试编写
  - 代码实现
  - 测试通过
- **版本控制**
  - 合并分支



### 感谢

本书能够出现，要感谢很多人。这其中有很多人的文章及作品为本书的书写提供了充实的资料和参考，有些人默默付出，让我能够安心的完成本书！这其中有我的家人也有志同道合的朋友！

- [Rails官网](https://rubyonrails.org/) : 十分感激创造出Rails这么优秀的框架，让我感受的语言的优美，编程的快乐。这里有最新的信息和文档，希望大家有时间多逛逛这里。
- [ruby中国](https://ruby-china.org/) : 这是一个超级友好的社区，里面没有撕逼，只有互相共勉，互相支持，希望更多的人可以喜欢这个社区，我非常感谢这个社区，在这里能够感受到技术人的热情与执着。这个社区还维护着国内的稳定的gem源，感谢作者及团队的默默付出！
- [Alexandre Rousseau](https://github.com/madeindjs) ：十分感谢该作者的文献，该作者的文档写的极具水平，能把知识溶于简单的语言，十分感激与敬佩。
- [theforeman](https://github.com/theforeman) ：这是一只十分优秀的团队，一直在维护着十分优秀的项目！本书代码的一些灵感来源于该团队的开源项目，十分感激！



###  本书参与者【排名不分先后】

| 等你哦伙伴++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ |
| ------------------------------------------------------------ |
| <img src="https://github.com/letseeqiji/pure-rails6-api/blob/main/resources/avatar/Funggg.png?raw=true" alt="avatar" />   [TerryFunggg](https://github.com/TerryFunggg) |
| ![avatar](https://github.com/letseeqiji/pure-rails6-api/blob/main/resources/avatar/kba977.png?raw=true)   [kba977](https://github.com/kba977) |
| <img src="https://github.com/letseeqiji/pure-rails6-api/blob/main/resources/avatar/Kaizhao%20Zhang.png?raw=true" alt="avatar"  />   [Kaizhao Zhang](https://github.com/zhangkaizhao)      |
| <img src="https://raw.githubusercontent.com/letseeqiji/pure-rails6-api/main/resources/avatar/VictorNanka.png" alt="avatar"  />   [VictorNanka](https://github.com/VictorNanka)      |
| <img src="https://github.com/letseeqiji/pure-rails6-api/blob/main/resources/avatar/qijige.png?raw=true" alt="avatar"  />   [letseeqiji](https://github.com/letseeqiji)                                              |

---



### 期待

期待本书中的内容可以为读者提供有意义的指导，如果阅读中遇到什么问题，或者有好的建议，您可以发送邮件到我的邮箱： wowiwo@yeah.net

最后，期待我们一起奋力前行，让努力的人不孤单！加油，伙伴们！！！
