---
layout: post
title:  "亚马逊架构简介"
date:   2016-2-28 00:00:00
categories: architecture
author: Paul Huang
---

This is a wonderfully informative Amazon update based on Joachim Rohde's discovery of an interview with Amazon's CTO. You'll learn about how Amazon organizes their teams around services, the CAP theorem of building scalable systems, how they deploy software, and a lot more. Many new additions from the ACM Queue article have also been included.

这是一篇关于亚马逊如何逐步升级的精彩短文，核心内容源自Joachim Rohde对亚马逊CTO的专访内容的分析
通过本文你将知道亚马逊如何以服务为中心组建他们的团队，了解用来构建可伸缩系统的CAP理论，以及他们如何部署软件等等。许多来自[ACM Queue](http://queue.acm.org/)的内容也被收入文中。

Amazon grew from a tiny online bookstore to one of the largest stores on earth. They did it while pioneering new and interesting ways to rate, review, and recommend products. Greg Linden shared is version of Amazon's birth pangs in a series of blog articles

亚马逊从一间线上书店起步，直至成为当今世上最大的在线零售商之一。
为此他们提供了很多先驱性的功能，例如最早提供了对商品的投票，预览和推荐功能。
对此，Greg Linden分享了一系列关于亚马逊初生阵痛的博文。

![Amazon](/assets/images/posts/amazon-logo.jpg)

<!--more-->

## 平台
- Linux
- Oracle
- C++
- Perl
- Mason
- Java
- Jboss
- Servlets

## 目前状态

- More than 55 million active customer accounts.
- 超过5500万个活跃账号

- More than 1 million active retail partners worldwide.
- 超过100万个来自世界各地的零售合作伙伴

- Between 100-150 services are accessed to build a page.
- 接入100-150种左右的服务以构建页面

## 产品架构

- What is it that we really mean by scalability? A service is said to be scalable if when we increase the resources in a system, it results in increased performance in a manner proportional to resources added. Increasing performance in general means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.

- 可伸缩性真正的含义是什么？就服务而言，可伸缩性意味着，当我们为系统增加资源的时，系统性能获得相应比例的提升。而性能提升意味着能为更多客户提供服务，或者说能在客户不断成长时满足他们的需要。

- The big architectural change that Amazon made was to move from a two-tier monolith to a fully-distributed, decentralized, services platform serving many different applications.

- 对于亚马逊而言，一个大的架构改动是将系统从二层架构转变为分布式服务，并让各个服务提供不同应用。

- Started as one application talking to a back end. Written in C++.
让我们从一个由C++编写的后端应用开始讲起。

- It grew. For years the scaling efforts at Amazon focused on making the back-end databases scale to hold more items, more customers, more orders, and to support multiple international sites. In 2001 it became clear that the front-end application couldn't scale anymore. The databases were split into small parts and around each part and created a services interface that was the only way to access the data.

- 多年来，为了能够支持更多商品，更多客户，更多订单以及为更多国际站点提供服务，这个应用不断变大，于此同时亚马逊也将扩展系统的工作重点也被放在了扩展后台数据库上。但是到2001年，大家发现，这个后台应用已经无法进行任何扩展了。数据库已经被切割为多个部分，并且，各部分都建立了相应的服务接口来存取数据。

- The databases became a shared resource that made it hard to scale-out the overall business. The front-end and back-end processes were restricted in their evolution because they were shared by many different teams and processes.

- 由于数据库是共享资源，因此很难再扩展业务。同时，由于修改业务流程会牵涉多个不同的团队，因此前后端的流程改进受到了严格的限制。

- Their architecture is loosely coupled and built around services. A service-oriented architecture gave them the isolation that would allow building many software components rapidly and independently.

- 为了解决这个问题，他们基于服务建立了松耦合的架构。面向服务的架构使他们能保证各模块相互独立，从而允许快速独立的搭建更多软件组件。

- Grew into hundreds of services and a number of application servers that aggregate the information from the services. The application that renders the Amazon.com Web pages is one such application server. So are the applications that serve the Web-services interface, the customer service application, and the seller interface.

- 在此基础上，亚马逊开发了数以百计的服务和大量应用用来汇总信息。例如，绘制Amazon.com页面的就是其中一个应用。这些应用还提供了web服务接口，客户服务应用及销售接口

- Many third party technologies are hard to scale to Amazon size. Especially communication infrastructure technologies. They work well up to a certain scale and then fail. So they are forced to build their own.
Not stuck with one particular approach. Some places they use jboss/java, but they use only servlets, not the rest of the J2EE stack.

- 由于亚马逊巨大的规模，许多第三方技术难以扩展与之相适应。以通信设施的技术来说，系统扩展他到某种规模以后现有方案完全不能正常工作。因此亚马逊被建立自己的技术用以能满足各种应用场景。一些情况下他们使用jboss/java，但仅使用servlets,而不包括其他J2EE技术栈。


- C++ is uses to process requests. Perl/Mason is used to build content.
Amazon doesn't like middleware because it tends to be framework and not a tool. If you use a middleware package you get lock-in around the software patterns they have chosen. You'll only be able to use their software. So if you want to use different packages you won't be able to. You're stuck. One event loop for messaging, data persistence,
AJAX, etc. Too complex. If middleware was available in smaller components, more as a tool than a framework, they would be more interested.

- C++用来处理请求，Perl/Mason用来构建内容。亚马逊不喜欢中间件，因为它往往倾向于成为框架而不是工具。所以如果你使用中间件，你会发现自己被软件合作方绑住了手脚，你将只能使用他们的软件，而不能做出其他选择，总而言之，你会被困住。不过对于事件消息循环，数据持久化，AJAX等问题的处理技术太过复杂，因此，如果中间件对小型组建可用，更像工具而不是框架，那他们将会获得更多的关注。

- The SOAP web stack seems to want to solve all the same distributed systems problems all over again.
Offer both SOAP and REST web services. 30% use SOAP. These tend to be Java and .NET users and use WSDL files to generate remote object interfaces. 70% use REST. These tend to be PHP or PERL users.
In either SOAP or REST developers can get an object interface to Amazon. Developers just want to get job done. They don't care what goes over the wire.

- SOAP web栈看起想解决所有和分布式系统有关的问题。它同时提供SOAP和RESTweb服务。
30%使用SOAP，这部分倾向于Java或.NET使用者，因而使用WSDL文档去建立一个远程对象接口。
70%使用REST，这部分倾向于PHP或PERL使用者。
因此无论习惯于SOAP或REST服务的开发人员都能从亚马逊获取相应接口。要知道开发人员往往只想把工作搞定，他们才不关心电线里传输的是什么。

- Amazon wanted to build an open community around their services. Web services were chosed because it's simple. But hat's only on the perimeter. Internally it's a service oriented architecture. You can only access the data via the interface. It's described in WSDL, but they use their own encapsulation and transport mechanisms.

- 亚马逊希望能围绕他们的服务建立一个社区，而之所以采用Web服务仅仅是因为它足够简单。但帽子永远只能看到外沿，就内部而言，这是一个以服务为导向的架构。你能通过接口获取数据，接口通过WSDL描述，但他们使用用自己特有的封装和传输机制。

- Teams are Small and are Organized Around Services
  团队很小而且紧紧围绕服务组建

    - Services are the independent units delivering functionality within Amazon. It's also how Amazon is organized internally in terms of teams.

    - 亚马逊内部，服务作为独立模块用来提供功能，同时这也是亚马逊内部组建团队的方式。

    - If you have a new business idea or problem you want to solve you form a team. Limit the team to 8-10 people because communication hard. They are called two pizza teams. The number of people you can feed off two pizzas.

    - 例如，你想到一个新的商业计，同时你想组建一个团队，那么请把人数控制在8-10人左右，因为人数过多会导致沟通困难。这个团队又被称作2饼小组（two pizza teams），因为只要2张大饼（pizza）就能喂饱他们了：）

    - Teams are small. They are assigned authority and empowered to solve a problem as a service in anyway they see fit.

    - 因为团队很小，所以他们才能被授权去解决任何发现的问题

    - As an example, they created a team to find phrases within a book that are unique to the text. This team built a separate service interface for that feature and they had authority to do what they needed.

    - 举例来说，亚马逊建立一个团队去查找一本书中的特有短句。这个团队为实现这个功能搭建了一个独立的服务接口来保证他们拥有任何他们想要的权限

    - Extensive A/B testing is used to integrate a new service . They see what the impact is and take extensive measurements.

    - 大量的[A/B测试](http://baike.baidu.com/view/4357479.htm) 被用来集成新服务。这样他们才能通过广泛的观测了解他么的改动会产生何种影响。

- Deployment
- 部署

    - They create special infrastructure for managing dependencies and doing a deployment.

    - 亚马逊建立了专用设施来管理依赖和执行部署。

    - Goal is to have all right services to be deployed on a box. All application code, monitoring, licensing, etc should be on a box.

    - 主要目标是能找到相应的服务并打包部署。所有的代码，监视器，授权许可等都会被打包。

    - Everyone has a home grown system to solve these problems.

    - 并而保证每个人都能有一个定制系统去解决这些问题。

    - Output of deployment process is a virtual machine. You can use EC2 to run them.

    - 部署流程改进后的最终产物是一台虚拟机。而现在，你能用EC2去运行它们


- Work From the Customer Backwards to Verify a New Service is Worth Doing

- 以客户为导向从而确认新功能（服务）的价值

    - Work from the customer backward. Focus on value you want to deliver
    for the customer.

    - 亚马逊提倡以客户为导向展开工作，专注于为客户提供价值

    - Force developers to focus on value delivered to the customer instead of building technology first and then figuring how to use it.

    - 这意味着需要严格要求开发人员专注于为客户提供价值，而不是先考虑如何开发新技术，再考虑在哪些情况下使用它。

    - Start with a press release of what features the user will see and work backwards to check that you are building something valuable.

    - 所有工作应该从一篇用户会得到何种功能的通讯稿开始，然后反向推导校验开发工作的价值

    - End up with a design that is as minimal as possible. Simplicity is the key if you really want to build large distributed systems.

    - 同时完成的设计必须尽可能的简单的。如果你希望构建一个大型分布式系统，极简主义将成为成功的关键

- State Management is the Core Problem for Large Scale Systems

- 状态管理是大型可伸缩系统的核心问题

    - Internally they can deliver infinite storage.

    - 就系统内部而言，你或许能部署一个无限大的存储系统用来保存状态，但大部分情况并非如此。

    - Not all that many operations are stateful. Checkout steps are stateful.

    - 对于状态管理，首先要知道的一点是：不是所有的操作都是有状态的，请先找出那些包含状态的步骤。

    - Most recent clicked web page service has recommendations based on session IDs.

    - 举例来说：展示最近打开的页面（包含推荐页面）这一功能就基于session ID的

    - They keep track of everything anyway so it's not a matter of keeping state. There's little separate state that needs to be kept for a session. The services will already be keeping the information so you just use the services.

    - 因为想要保存这个状态，意味着要需要持续跟踪记录用户的所有动作，因此几乎无法在浏览器实现。所以这个状态就需要保存到session中。现在服务器已经保存了这个信息，如果有需要，你可以直接调用服务。

- Eric Brewer's CAP Theorem or the Three properties of Systems
- Eric Brewer的CAP理论

    - Three properties of a system: consistency, availability, tolerance to network partitions.

    - CAP中的三个属性分别为： 一致性，可用性，分区容错性

    - Consistency: write a value and then you read the value you get the same value back. In a partitioned system there are windows where that's not true.

    - 数据一致性：等同于所有节点访问同一份最新的数据副本

    - Availability: may not always be able to write or read. The system will say you can't write because it wants to keep the system consistent.

    - 可用性：对数据更新具备高可用性

    - Partitionability: divide nodes into small groups that can see other groups, but they can't see everyone.

    - 分区容错性：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择

    - You can have at most two of these three properties for any shared-data system.

    - 理论的核心是：在任何共享数据的系统中，你最多能完全满足其中两点。原因见[这里](https://www.zybuluo.com/jewes/note/68185)

    - To scale you have to partition, so you are left with choosing either high consistency or high availability for a particular system. You must find the right overlap of availability and consistency.

    - 就实际情况而言，为了满足可伸缩性，必须让系统支持分区，因此你必须为你的系统找到一个一致性和可用性平衡点

    - Choose a specific approach based on the needs of the service.

    - 简而言之，就是根据所需服务选择合适的方案

    - For the checkout process you always want to honor requests to add items to a shopping cart because it's revenue producing. In this case you choose high availability. Errors are hidden from the customer and sorted out later.

    - 对于结账流程为例：你经常会遇到这样的场景，购物车结算后发现满足奖励条件，于是用户打算将奖励商品添加到购物车中。在这个流程中，往往会选择高可用性。而数据一致性的问题会被推迟到客户完成商品挑选的流程之后再去解决。

    - When a customer submits an order you favor consistency because several services--credit card processing, shipping and handling, reporting--are simultaneously accessing the data.
    - 而当客户提交订单后，你会希望保证数据的一致性，因为 信用卡处理，物流，报表等服务都对数据有严格要求


## 新技能get
- You must change your mentality to build really scalable systems. Approach chaos in a probabilistic sense that things will work well. In traditional systems we present a perfect world where nothing goes down and then we build complex algorithms (agreement technologies) on this perfect world. Instead, take it for granted stuff fails, that's
reality, embrace it. For example, go more with a fast reboot and fast recover approach. With a decent spread of data and services you might get close to 100%. Create self-healing, self-organizing lights out operations.

- 要建立真正的可伸缩系统，你必须首先转变思维模式。从概率上来说，即使方案不太靠谱，产品依然能正常工作。在传统思维里，我们希望展现一个不断进步系统，因此在这个完美系统里，我们需要建立及其复杂的算法（技术协议）来解决问题。相反，真实世界里，错误随处可见。因此，构建一个能快速重启和恢复的方案，同时在服务器间可靠的传递数据，使用自我恢复，自我管理的轻量操作，你的目标就接近100%达成了。

- Create a shared nothing infrastructure. Infrastructure can become a shared resource for development and deployment with the same downsides as shared resources in your logic and data tiers. It can cause locking and blocking and dead lock. A service oriented architecture allows the creation of a parallel and isolated development process that scales feature development to match your growth.

- 请建立一个对其他系统没有依赖的基础系统。基础系统共享资源或许在开发和部署是一个优点，但在逻辑层和数据层来看，这也是一个缺点。因为这可能导致锁定和死锁。一个面向服务的架构应该允许某种程度的并发，并隔离开发进程，只有这样，你才能在系统不断的增长时拥有可伸缩的特性。

- Open up you system with APIs and you'll create an ecosystem around your application.

- 开放你的API，你将会有一个围绕你的应用建立的生态。

- Only way to manage as large distributed system is to keep things as simple as possible. Keep things simple by making sure there are no hidden requirements and hidden dependencies in the design. Cut technology to the minimum you need to solve the problem you have. It doesn't help the company to create artificial and unneeded layers of complexity.

- 管理大型分布式系统的唯一方法就是让每件事尽可能简单。保持简单意味着确保在你的设计中不包含任何隐藏的需求和依赖。用尽可能少的技术来解决你的问题。对公司来说刻意构建的复杂架构往往没有意义。

- Organizing around services gives agility. You can do things in parallel is because the output is a service. This allows fast time to market. Create an infrastructure that allows services to be built very fast.

- 围绕服务组织资源让开发变得更敏捷。当最终成果时一个服务的时候，你能同步实现多个目标。这就让更快销售成为可能。建立一个基础服务让服务构建变得更迅速

- There's bound to be problems with anything that produces hype before real implementation
- 在动手开发之前，一定会过度考虑某些问题。

- Use SLAs internally to manage services.
- 在内部使用[SLA](http://baike.baidu.com/view/163802.htm)来管理服务。

- Anyone can very quickly add web services to their product. Just implement one part of your product as a service and start using it.
- 确保任何人都能快速将web服务添加到他们的产品中。每次仅仅实现你产品的一小部分并且立刻开始使用它。

- Build your own infrastructure for performance, reliability, and cost control reasons. By building it yourself you never have to say you went down because it was company X's fault. Your software may not be more reliable than others, but you can fix, debug, and deployment much quicker than when working with a 3rd party.
- 基于性能，可靠性和成本控制等原因，你应该建立自己的基础设施。假如一切都是你亲手搭建的，那你就没有机会说：这个问题是X公司的产品造成的。你的软件将变得比其它人的更可靠，因为修复，调试，部署的权利都掌握在你自己手里。

- Use measurement and objective debate to separate the good from the bad. I've been to several presentations by ex-Amazoners and this is the aspect of Amazon that strikes me as uniquely different and interesting from other companies. Their deep seated ethic is to expose real customers to a choice and see which one works best and to make decisions based on those tests.
- 效果观测和目标讨论有助于区分设计高下。前亚马逊员工已经数次在我面前展示了这些，正是这些让亚马逊具备其它公司所没有的吸引力。他们根深蒂固的文化就是把产品交给市场，让市场来做出决定。

- Avinash Kaushik calls this getting rid of the influence of the HiPPO's, the highest paid people in the room. This is done with techniques like A/B testing and Web Analytics. If you have a question about what you should do code it up, let people use it, and see which alternative gives you the results you want.
- Avinash Kaushik,这个房间里收入最高的人，管这个叫消除河马的影响力。这情况一般发生在技术讨论时，而且和A/B测试还有网页分析一起发生。惯例就是，如果你有问题，那么你就该把代码写出来，让客户来用它，然后看他们的选择是否和你一致。

- Create a frugal culture. Amazon used doors for desks, for example.
- 建立一种朴素的文化。举例来说，亚马逊就把门板拿来当桌子。

- Know what you need. Amazon has a bad experience with an early recommender system that didn't work out: "This wasn't what Amazon needed. Book recommendations at Amazon needed to work from sparse data, just a few ratings or purchases. It needed to be fast. The system needed to scale to massive numbers of customers and a huge catalog. And it needed to enhance discovery, surfacing books from deep in the catalog that readers wouldn't find on their own."
- 了解你所需要的内容。关于早期的推荐系统，亚马逊有一个负面经验，因为它根本不能正常工作。"这不是亚马逊想要的结果。它导致亚马逊的书本推荐只拥有很少的基础数据，零星的打分和购买量。系统需要足够快，需要可以扩展以应对海量客户和巨大的图书分类。因此需要加强检索，帮助读者从数据中挖掘出他们没有的商品"

- People's side projects, the one's they follow because they are interested, are often ones where you get the most value and innovation. Never underestimate the power of wandering where you are most interested.

- 大家看来很支持这个项目。通常大家关注某个项目仅仅是因为喜欢，而最能给人带来价值和耳目一新的感觉的项目往往最受关注。不要低估兴趣的力量。

- Involve everyone in making dog food. Go out into the warehouse and pack books during the Christmas rush. That's teamwork.

- 请大家吃个便饭，然后在黑五期间把书打包外出找个仓库围在一起，这才是真正的团队工作。

- Create a staging site where you can run thorough tests before releasing into the wild.

- 建立一个框架站点，保证你能在结果发布前在上面运行测试。

- A robust, clustered, replicated, distributed file system is perfect for read-only data used by the web servers.

- 一个强壮的，集群式的，可复用的，分布式的文件系统特别适合为web服务提供只读数据

- Have a way to rollback if an update doesn't work. Write the tools if necessary.

找到一种方法在方案不工作的时候回滚它。如果必要，甚至应该专门写一个工具

- Switch to a deep services-based architecture (http://webservices.sys-con.com/read/262024.htm).

- 切换到[基于服务的架构](http://webservices.sys-con.com/read/262024.htm)

- Look for three things in interviews: enthusiasm, creativity, competence. The single biggest predictor of success at Amazon.com was enthusiasm.

- 在面试的时候观察三种能力：热情，创造力，能力。将亚马逊今日成就的寓言变为现实的就是热情

- Hire a Bob. Someone who knows their stuff, has incredible debugging skills and system knowledge, and most importantly, has the stones to tackle the worst high pressure problems imaginable by just leaping in.

- 雇一个乔布斯，这个人对别人的工作了如指掌，具有超凡的调试技巧和系统知识，并且自命不凡，能立刻接手最难缠，最烫手的问题

- Innovation can only come from the bottom. Those closest to the problem are in the best position to solve it. any organization that depends on innovation must embrace chaos. Loyalty and obedience are not your tools.

- 革新只能自下而上，那些最接近问题的人才拥有最好的位置去解决它。水至清则无鱼，任何希望革新的组织都必须接受某种程度的混乱，恭顺并不能成为你的工具

- Creativity must flow from everywhere.
- 在任何场合，创造力都不应被压制

- Everyone must be able to experiment, learn, and iterate. Position, obedience, and tradition should hold no power. For innovation to flourish, measurement must rule.

- 每个人都必须通过实践、学习和迭代来获得成长，地位、服从和传统都应该让位。相较华丽的革新，可度量的结果才应该成为唯一准则

- Embrace innovation. In front of the whole company, Jeff Bezos would give an old Nike shoe as "Just do it" award to those who innovated.

- 拥抱革新。在公司所有人面前，Jeff Bezos将会颁发一双旧Nike作为革新者"Just do it"的奖励

- Don't pay for performance. Give good perks and high pay, but keep it flat. Recognize exceptional work in other ways. Merit pay sounds good but is almost impossible to do fairly in large organizations. Use non-monetary awards, like an old shoe. It's a way of saying thank you, somebody cared.

- 不要将绩效作为收入标准，而应该将高薪和奖励平均分配给所有人。应该通过其它手段分辨出特别的工作。绩效公司或许听起来不错但在巨型组织内部几乎不可能保证公平。使用非货币的奖励，比如一双旧鞋。这是一种表达关爱的更好的手段

- Get big fast. The big guys like Barnes and Nobel are on your tail. Amazon wasn't even the first, second, or even third book store on the web, but their vision and drive won out in the end.

- 快速成长，因为马老板们很可能就在你后面。当年亚马逊或许连排名第三的书店都不如，但它的目标和方向却一直是在终点线胜出

- In the data center, only 30 percent of the staff time spent on infrastructure issues related to value creation, with the remaining 70 percent devoted to dealing with the "heavy lifting" of hardware procurement, software management, load balancing, maintenance, scalability challenges and so on.

- 在数据中心，仅30%的工作时间被用在解决和创造价值上，而其余70%都被分配到买硬件，软件管理，负载均衡，维护等这样的"扔铁"项目上。

- Prohibit direct database access by clients. This means you can make you service scale and be more reliable without involving your clients. This is much like Google's ability to independently distribute improvements in their stack to the benefit of all applications.

- 禁止客户端直接存取数据。因为这意味着你的服务更具伸缩性和可靠性。就像谷歌，通过独立分散的改进不断叠加来增加所有应用的价值

- Create a single unified service-access mechanism. This allows for the easy aggregation of services, decentralized request routing, distributed request tracking, and other advanced infrastructure techniques.

- 建立一个独立统一的服务获取机制。这将保证服务更容易被集成，去中心化的请求路由，分布式的请求跟踪以及其他高级的基础技术

- Making Amazon.com available through a Web services interface to any developer in the world free of charge has also been a major success because it has driven so much innovation that they couldn't have thought of or built on their own.

- 让亚马逊的web服务接口对所有开发人员免费开放已经成为了另一个巨大的成功，因为这导致了如此之多的革新，连当初构建这个服务的人都想象不到

- Developers themselves know best which tools make them most productive and which tools are right for the job.
- 开发人员自己知道对于解决问题来说最具效率的工具是什么

- Don't impose too many constraints on engineers. Provide incentives for some things, such as integration with the monitoring system and other infrastructure tools. But for the rest, allow teams to function as independently as possible.
- 不要为工程师增加太多束缚。为某个目标提供动机，例如需要将其整合到监视系统和其他基础工具中，就其他方面而言，应该允许开发组尽可能独立的运作。

- Developers are like artists; they produce their best work if they have the freedom to do so, but they need good tools. Have many support tools that are of a self-help nature. Support an environment around the service development that never gets in the way of the development itself.

- 开发人员就像艺术家，当他们获得自由的时候他们会创造出最好的作品，但是他们需要好的工具，有许多好的工具都具有自助性。因此，为服务的开发环境提供一个良好的生态对开发人员相当重要

- You build it, you run it. This brings developers into contact with the day-to-day operation of their software. It also brings them into day-to-day contact with the customer. This customer feedback loop is essential for improving the quality of the service.

- 谁开发，谁运行。这将让开发人员日复一日的去使用他们自己的产品。而这也将把他们和使用他们产品的客户紧密联系到一起，客户的反馈最终会让他们提高产品的质量

- Developers should spend some time with customer service every two years. Their they'll actually listen to customer service calls, answer customer service e-mails, and really understand the impact of the kinds of things they do as technologists.

- 每两年，开发人员都应该有机会在客服岗位上体验工作。那样，他们就将有就会听取客服录音，回答客户邮件，切身了解他们作为技术人员所能产生的影响

- Use a "voice of the customer," which is a realistic story from a customer about some specific part of your site's experience. This helps managers and engineers connect with the fact that we build these technologies for real people. Customer service statistics are an early indicator if you are doing something wrong, or what the real pain points are for your customers.

倾听"客户的声音"。一个关于网站某些功能的真实用户感受，能帮助管理层和工程师了解他们的工作的意义。客户服务统计是发现你走错方向的探测器，也是发现客户痛点的最佳方法之一

- Infrastructure for Amazon, like for Google, is a huge competitive advantage. They can build very complex applications out of primitive services that are by themselves relatively simple. They can scale their operation independently, maintain unparalleled system availability, and introduce new services quickly without the need for massive reconfiguration.

- 亚马逊的基础设施就和谷歌一样，拥有巨大的竞争优势。因为简单，他们能通过非常原始的服务建立非常复杂的应用。它们能以独立的扩展操作，可以维护具有超高可用性的系统，并且能不修改任何配置，直接接入新服务

## 参考文章
- [Amazon Architecture](http://highscalability.com/amazon-architecture)
- [Early Amazon by Greg Linden](http://glinden.blogspot.jp/2006/05/early-amazon-end.html)
- [How Linux saved Amazon millions](http://news.com.com/2100-1001-275155.html)
- [Interview Werner Vogels - Amazon's CTO](http://www.se-radio.net/?post_id=157593)
- [Asynchronous Architectures - a nice summary of Werner Vogels' talk by Chris Loosley](http://www.webperformancematters.com/journal/2007/8/21/asynchronous-architectures-4.html)
- [Learning from the Amazon technology platform - A Conversation with Werner Vogels](http://queue.acm.org/detail.cfm?id=1142065)
- [Werner Vogels' Weblog - building scalable and robust distributed systems](http://www.allthingsdistributed.com/)
