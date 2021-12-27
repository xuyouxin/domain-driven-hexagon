# 领域驱动的六边形架构

这个项目的要点是提供有关如何设计应用程序的建议。这个 README 文档介绍了从不同来源收集到的一些技术、工具、最佳实践、架构模式和指南。

以下的所有信息都应该被视为建议。请记住不同项目的需求不同，因此如果需要可以替换或者直接跳过本 README 文档中提到的任何模式。

代码示例使用 NodeJS， TypeScript， NestJS 框架并使用了 Typeorm 来访问数据库。

尽管这里的模式和原则是框架/语言无关的，因此上述技术可以被任何替代方案取代。无论使用什么语言或框架，任何应用程序都可以从下面描述的原则中受益。

注意：代码示例适用于 TypeScript 和上述框架，因此可能不适用于其他语言。还要记住的是这里的代码示例仅仅是示例，实际中应该根据项目需要和个人喜好进行更改。

## 目录

- [架构(Architecture)](#架构(Architecture))

  - [图解(Diagram)](#图解(Diagram))
  - [模块(Modules)](#模块(Modules))
  - [应用程序核心(Application Core)](#应用程序核心(Application Core))
  - [应用层(Application layer)](#应用层(Application layer))
    - [应用服务(Application Services)](#应用服务(Application Services))
    - [命令和查询(Commands and Queries)](#命令和查询(Commands and Queries))
    - [端口(Ports)](#端口(Ports))
  - [领域层(Domain Layer)](#领域层(Domain Layer))
    - [实体(Entities)](#实体(Entities))
    - [聚合(Aggregates)](#聚合(Aggregates))
    - [领域事件(Domain Events)](#领域事件(Domain Events))
    - [集成事件(Integration Events)](#集成事件(Integration Events))
    - [领域服务(Domain Services)](#领域服务(Domain Services))
    - [值对象(Value Objects)](#值对象(Value Objects))
    - [强制领域对象的有效性(Enforcing invariants of Domain Objects)](#强制领域对象的有效性(Enforcing invariants of Domain Objects))
    - [领域错误(Domain Errors)](#领域错误(Domain Errors))
    - [在应用核心内部使用库(Using libraries inside application's core)](#在应用核心内部使用库(Using libraries inside application's core))
  - [接口适配器(Interface Adapters)](#接口适配器(Interface Adapters))
    - [控制器(Controllers)](#控制器(Controllers))
    - [数据传输对象(DTOs)](#数据传输对象(DTOs))
  - [基础设施(Infrastructure)](#基础设施(Infrastructure))
    - [适配器(Adapters)](#适配器(Adapters))
    - [存储库(Repositories)](#存储库(Repositories))
    - [持久化模型(Persistence models)](#持久化模型(Persistence models))
    - [其他可以作为基础设施层一部分的东西(Other things that can be a part of Infrastructure layer)](#其他可以作为基础设施层一部分的东西(Other things that can be a part of Infrastructure layer))
  - [对小型APIs的建议(Recommendations for smaller APIs)](#对小型APIs的建议(Recommendations for smaller APIs))
  - [架构、最佳实践、设计模式和原则的通用建议(General recommendations on architectures, best practices, design patterns and principles)](#架构、最佳实践、设计模式和原则的通用建议(General recommendations on architectures, best practices, design patterns and principles))

- [其他建议和最佳实践(Other recommendations and best practices)](#其他建议和最佳实践(Other recommendations and best practices))

  - [异常处理(Exceptions Handling)](#异常处理(Exceptions Handling))
  - [测试(Testing)](#测试(Testing))
    - [压力测试(Load Testing)](#压力测试(Load Testing))
    - [模糊测试(Fuzz Testing)](#模糊测试(Fuzz Testing))
  - [配置(Configuration)](#配置(Configuration))
  - [日志(Logging)](#日志(Logging))
  - [健康监测(Health monitoring)](#健康监测(Health monitoring))
  - [目录和文件结构(Folder and File Structure)](#目录和文件结构(Folder and File Structure))
  - [文件名(File names)](#文件名(File names))
  - [静态代码分析(Static Code Analysis)](#静态代码分析(Static Code Analysis))
  - [代码格式化(Code formatting)](#代码格式化(Code formatting))
  - [文档(Documentation)](#文档(Documentation))
  - [使应用程序易于设置(Make application easy to setup)](#使应用程序易于设置(Make application easy to setup))
  - [种子数据(Seeds)](#种子数据(Seeds))
  - [数据迁移(Migrations)](#数据迁移(Migrations))
  - [限速(Rate Limiting)](#限速(Rate Limiting))
  - [代码生成(Code Generation)](#代码生成(Code Generation))
  - [自定义工具类型(Custom utility types)](#自定义工具类型(Custom utility types))
  - [预推送/预提交钩子(Pre-push/pre-commit hooks)](#预推送/预提交钩子(Pre-push/pre-commit hooks))
  - [避免大规模继承链(Prevent massive inheritance chains)](#避免大规模继承链(Prevent massive inheritance chains))
  - [提交规范(Conventional commits)](#提交规范(Conventional commits))

- [其他资源(Additional resources)](#其他资源(Additional resources))
  - [文章(Articles)](#文章(Articles))
  - [GitHub仓库(Github Repositories)](#GitHub仓库(Github Repositories))
  - [文档站点(Documentation websites)](#文档站点(Documentation websites))
  - [博客(Blogs)](#博客(Blogs))
  - [视频(Videos)](#视频(Videos))
  - [书籍(Books)](#书籍(Books))

# 架构(Architecture)

这部分主要基于：

- [Domain-Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design)
- [Hexagonal (Ports and Adapters) Architecture](https://blog.octo.com/en/hexagonal-architecture-three-principles-and-an-implementation-example/)
- [Secure by Design](https://www.manning.com/books/secure-by-design)
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Onion Architecture](https://herbertograca.com/2017/09/21/onion-architecture/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Software Design Patterns](https://refactoring.guru/design-patterns/what-is-pattern)

还有很多其他资源（下面的各章节中的链接）。

在我们开始之前，先看一下使用完整架构的利弊：

#### 优点

- 独立于外部框架、技术、数据库等。框架和外部资源可以被以较低的成本被引入和移除。
- 易于测试和扩展。
- 更安全。一些安全原则天然地存在于设计中。
- 解决方案可以被不同的团队开发和维护，而且不会相互制约。
- 易于增加新特性。随着系统随时间推移而发展，增加新功能的难度能保持在一个较小的水准。
- 如果解决方案能被正确地分解 [边界上下文(bounded context)](https://martinfowler.com/bliki/BoundedContext.html)，在需要时很容易将它的一部分转换成微服务。

#### 缺点

- 这是一个复杂的架构，它需要我们对软件的质量原则有深刻的理解，例如 SOLID、整洁/六边形架构、领域驱动设计等。任何实施此类解决方案的团队几乎肯定需要一个专家来驱动解决方案并防止以错误的方式发展和积累技术债。

- 对于业务逻辑较少的中小型规模的应用程序，不太推荐使用这里描述的一些实践。因为为了支持构建块和层、样板代码、抽象、数据映射等增加了前期复杂性。因此，像实现这样一个完整架构通常不太适用于简单的应用 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 并可能导致解决方案过于复杂。下面描述的一些原则可以被用于小规模的应用程序，但请在分析和理解了所有利弊之后再使用。

# 图解(Diagram)

![领域驱动的六边形架构](assets/images/DomainDrivenHexagon.png)
<sup>该图解主要基于 [这个](https://github.com/hgraca/explicit-architecture-php#explicit-architecture-1) + 其他在网上搜集的</sup>

简言之，数据流看起来像这样（从左至右）：

- 使用数据传输对象(DTO)将请求/命令行(Request/CLI)和命令/事件(command/event)发送给控制器(controller)。
- 控制器解析数据传输对象(DTO)，将它映射成一个命令/查询(Command/Query)对象，并将它传递给应用服务(application service)。
- 应用服务处理这个命令/查询(Command/Query)；它使用领域服务和/或实体执行业务逻辑，并通过端口(ports)使用基础设施层。
- 基础设施层(infrastructure layer)使用映射器(mapper)将数据转换成它需要的形式，使用存储库(repositories)获取/持久化数据，使用适配器(adapters)发送事件或进行其他I/O通信，将数据映射回领域对象的格式并传回应用服务层。
- 应用服务层处理完成后，它将数据/确认返回给控制器。
- 控制器将数据返回给用户(如果应用有展示层/试图，则返回给它们)。

每一层负责它自己的逻辑且拥有构建块，这些构建块通常应该在可能且有意义的情况下遵循 [Single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) （比如，只使用`存储库`访问数据库，使用`实体`处理业务逻辑等）。

**请记住** 不同项目可以在这里描述的步骤/层/构建块的基础上上做适当增减。如果应用程序需要就添加，如果应用程序不那么复杂且不需要所有抽象则跳过一些。

对任何项目都通用建议：分析应用程序的规模和复杂度，找到一个折中点并且根据项目需要使用尽可能多的层和构建块，并跳过那些可能会使项目过于复杂的东西。

下面详细描述每个细节。

# 模块(Modules)

这个项目的代码示例划分了模块（也叫组件）。每个模块的名称反映了领域中的一个重要概念，并拥有自己的目录和专用代码，且模块中的每个用例都拥有专属的目录来存放和它需要的大部分内容（这也被称作_垂直切片_）。

如果那些在逻辑上紧密联系的事物在物理上也彼此靠近，那么我们可以较容易地改变它们。将模块看做是一个将相关业务逻辑组合在一起的“盒子”。

尽量不要在模块和用例之间创建依赖关系，将公共逻辑移动到一个独立的文件中并让两者都依赖公共文件而不是相互依赖。

尽量使每个模块独立并尽量减少模块间的交互。将没模块看做是一个受单一上下文限制的的迷你应用程序。尽量避免在模块间的直接导入（比如从另一个领域中导入服务），因为这会造成 [紧密耦合(tight coupling)](<https://en.wikipedia.org/wiki/Coupling_(computer_programming)>)。避免模块间耦合可以使用消息总线(message bus)来使模块之间相互通信，比如你可以使用命令总线(commands bus)或订阅其他模块发出的事件（关于事件和命令总线的更多信息请见下文）。

这种方法可以确保[松散耦合(loose coupling)](https://en.wikipedia.org/wiki/Loose_coupling)，并且，如果界限上下文被正确定义和设计，则在需要时每个模块都可以很容易地在不触及其他领域逻辑的情况下被分割成微服务。

阅读关于模块化编程优势的更多信息：

- [Modular programming: Beyond the spaghetti mess](https://www.tiny.cloud/blog/modular-programming-principle/).
- [What are Modules in Domain Driven Design?](https://www.culttt.com/2014/12/10/modules-domain-driven-design/)

每个模块都按以下描述的层分开。

# 应用程序核心(Application Core)

这是使用 [领域驱动设计构建块(DDD building blocks)](https://dzone.com/articles/ddd-part-ii-ddd-building-blocks) 构建的系统的核心。

**领域层(Domain layer)**：
- 实体(Entities)
- 聚合(Aggregates)
- 领域服务(Domain Services)
- 值对象(Value Objects)
- 领域错误(Domain Errors)

**应用层(Application layer)**:

- 应用服务(Application Services)
- 命令和查询(Commands and Queries)
- 端口(Ports)

_当需要时还会加入更多构建块。_

---

# 应用层(Application layer)

## 应用服务(Application Services)

也被称为“工作流服务”，“用例”，“交互者”等。这些服务编排了完成照客户端施加的命令所需的步骤。

- 通常用来编排外部世界如何与你的应用程序交互并执行终端用户所需的任务。
- 不包含特定领域的业务逻辑。
- 操作标量类型，将它们转换为领域类型。一个标量类型可以被视为领域模型位置的任何类型。这包括了原始类型和不属于领域的类型。
- 应用服务声明其为了执行领域逻辑需要的基础设施层的依赖（通过使用端口）。
- 用于通过端口从数据库/外部世界获取领域实体（或其他任何东西）
- 使用端口执行其他进程外部通信（比如发出事件、发送email等）。
- 在处理一个实体/聚合的情况下，直接执行它们的方法。
- 在处理多个实体/聚合的情况下，使用一个领域服务来编排它们。
- 通常是一个命令/查询处理程序。
- 不应依赖于其他应用服务因为这可能导致问题（比如循环依赖）。

每个用例一个应用服务被认为是一个好的实践。

<details>
<summary>什么是“用例”？</summary>

[wiki](https://en.wikipedia.org/wiki/Use_case):

> 在软件和系统工程中，用例是一系列动作或事件步骤，通常定义角色（在UML中被称为参与者）和系统之间的交互以实现目标。

简单来说，用例就是应用程序所需的操作列表。

---

</details>

示例文件: [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts)

更多有关应用服务的信息：

- [Domain-Application-Infrastructure Services pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-application-infrastructure-services-pattern.html)
- [Services in DDD finally explained](https://developer20.com/services-in-ddd-finally-explained/)

## 命令和查询(Commands and Queries)

这个原则被称为 [命令-查询分离(Command–Query Separation(CQS))](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)。只有有可能，方法应该被分为`命令`（状态更改操作）和`查询`（数据检索操作）。为了明确区分这两种操作类型，输入对象可以被表示为`命令`和`查询`。在数据传输对象(DTO)到达领域层之前，它会被转换成一个`命令/查询`对象。

### 命令(Commands)

`命令`是一个表示用户意图的对象，比如`CreateUserCommand`。它描述一个单一的动作（但不执行它）。

`命令`被用于状态更改操作，像创建新用户和将它保存到数据库。创建、更新和删除操作被认为是状态更改操作。

数据检索是`查询`的职责，所以`命令`方法不应该返回业务数据。

一些 CQS 纯粹主义者可能会说一个`命令`不应该返回任何东西。但你经常会至少需要一个被创建对象的ID以便之后访问它。为了实现这个目的你可以让客户端生成一个 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) (更多信息: [CQS与服务器生成的IDs(CQS versus server generated IDs)](https://blog.ploeh.dk/2014/08/11/cqs-versus-server-generated-ids/)).

尽管，违反这个规则并返回一些元数据，像被创建对象的`ID`，重定向链接、确认消息、状态或其他元数据是一种比遵循教条更实用的方式。

跨多个聚合的`命令`（或通过事件以及其他别的东西）做出的所有变更应该在一个单独的数据库事务中被保存（如果你正在使用单个数据库）。这意味着在一个单一进程中，一个到达你的应用程序的命令/请求应该**只执行一次** [事务操作(transactional operation)](https://en.wikipedia.org/wiki/Database_transaction) 来保存**所有**变更（或在失败时取消**所有**该命令/请求所作的变更）。我们应该这样做来确保一致性。为此你可以把数据库操作包装在一个事务中或使用像 [工作单元(Unit of Work)](https://java-design-patterns.com/patterns/unit-of-work/) 模式。示例：[create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) 注意它是如何从`this.unitOfWork`获取一个事务性的存储库的。

**注意**：`命令`和这个东西很相似但不相同：[命令模式(Command Pattern)](https://refactoring.guru/design-patterns/command)。网上有多个定义相似但略有不同的实现。

要执行命令你可以使用一个`命令总线`而不是直接导入一个服务。这将使命令调用者和接收者解耦，因此你可以在不产生耦合的情况下从任意地方发送命令。

示例文件：

- [create-user.command.ts](src/modules/user/commands/create-user/create-user.command.ts) - 一个命令对象
- [create-user.message.controller.ts](src/modules/user/commands/create-user/create-user.message.controller.ts) - 控制器使用命令总线执行命令命令。这将控制器和命令处理器解耦
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 一个命令处理器
- [command-handler.base.ts](src/libs/ddd/domain/base-classes/command-handler.base.ts) - 命令处理器基础类使用一个事务包装执行过程

阅读更多：

- [What is a command bus and why should you use it?](https://barryvanveen.nl/blog/49-what-is-a-command-bus-and-why-should-you-use-it)

### 查询(Queries)

`查询`和`命令`相似。它表示用户有想要寻找某物的意图且描述了如何去做。

`查询`用于检索数据并且不应该造成任何状态变更（例如写数据库和文件等）。

查询通常只是一个数据检索操作且不涉及业务逻辑；因此，如果需要，可以完全跳过应用层和领域层。但是，如果在返回查询响应（例如某些计算操作）之前必须应用一些额外的非状态更改逻辑，则可以放在应用/领域层中完成。

和命令相似，如果有需要，查询可以使用`查询总线`。通过这个方式你可以在任何地方查询任何内容而无需直接导入存储库以避免耦合。

示例文件：

- [find-users.query.ts](src/modules/user/queries/find-users/find-users.query.ts) - 查询对象
- [find-users.query-handler.ts](src/modules/user/queries/find-users/find-users.query-handler.ts) - 一个查询完全跳过应用/领域层的例子

---

通过强制使`命令`和`查询`分离，代码变得易于理解。一个改变状态，另一个只是检索数据。

此外，如果未来某一天有需要，从一开始就遵循 CQS 将有助于将写入和读取模型分离到不同的数据库 (CQRS)。

**注意**：这个 repo 使用 [NestJS CQRS](https://docs.nestjs.com/recipes/cqrs) 包来提供命令/查询总线。

阅读有关 CQS 和 CQRS 的更多信息：

- [Command Query Segregation](https://khalilstemmler.com/articles/oop-design-principles/command-query-segregation/).
- [Exposing CQRS Through a RESTful API](https://www.infoq.com/articles/rest-api-on-cqrs/)
- [What is the CQRS pattern?](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [CQRS and REST: the perfect match](https://lostechies.com/jimmybogard/2016/06/01/cqrs-and-rest-the-perfect-match/)

---

## 端口(Ports)

端口（用于驱动适配器）是定义契约的接口，这些契约必须由基础设施适配器实现，来实现一些技术细节相关而非业务细节相关的逻辑。端口是那些业务逻辑所不关心的技术细节的抽象。

在应用程序核心**依赖是向内的**。外层可以依赖内层，但内层永远不应该依赖外层。应用程序核心不应该依赖于框架或直接访问外部资源。任何对进程外的外部调用或从远程进程检索资源/数据都应该通过端口（接口）来进行，在基础设施层的某处创建实现类并注入到应用程序核心中([依赖注入(Dependency Injection)](https://en.wikipedia.org/wiki/Dependency_injection) 和 [依赖反转(Dependency Inversion)](https://en.wikipedia.org/wiki/Dependency_inversion_principle))。这可以使业务逻辑独立于技术细节，有助于测试，允许我们可以很容易地引入/移除/替换任何外部资源以达到程序模块化和 [松耦合(loosely coupled)](https://en.wikipedia.org/wiki/Loose_coupling)。

- 端口基本上是一个定义了应该做什么操作而不关心怎么做的接口。
- 可以创建端来对领域中的I/O操作、技术细节、侵入性库、遗留代码等进行抽象。
- 应该创建能满足领域需求的端口，而不是简单地模仿工具API。
- 模拟(mock)实现可以在测试时传入端口。模拟实现可以使你的测试更快且独立于环境。
- 在设计端口时，请记住 [接口隔离原则(Interface segregation principle)](https://en.wikipedia.org/wiki/Interface_segregation_principle)。在合理的情况下可以将大型接口拆分为较小的接口，但同样请注意在没有必要时不要过度设计。
- 端口还就可以帮助我们延迟决策。甚至可以在决定使用哪些技术（框架，数据库等）之前设计领域层。

**注意**：由于多数端口实现是在应用层中注入并执行的，因此应用层是一个存放这些端口的好地方。但有些时候领域层的业务逻辑依赖于执行某些外部资源，这种情况下端口可以被放在领域层中。

**注意**：在较小型的应用程序/APIs中创建端口会带来不必要的抽象，这可能会使此类解决方案过于复杂。对于这类应用程序，直接使用具体实现替代端口就足够了。请在使用这个模式之前分析利弊。

示例文件：

- [repository.ports.ts](src/libs/ddd/domain/ports/repository.ports.ts)
- [logger.port.ts](src/libs/ddd/domain/ports/logger.port.ts)

---

# 领域层(Domain Layer)

这层包含了应用程序的业务规则。

领域应该只使用领域对象，下面是一些重点。

## 实体(Entities)

实体是领域的核心。它们封装了企业的业务规则和属性。一个实体可以是一个拥有属性和方法的对象，或者也可以是一组数据结构和函数。

实体代表业务模型并表达特定模型具有的属性，它可以做什么，何时以及在什么条件下它可以做。业务模型的例子有用户、产品、预定，票据，钱包等。

实体必须始终保护它的 [不可变性(invariant)](https://en.wikipedia.org/wiki/Class_invariant):

领域实体应该总是有效的实体。一个对象有一定数量的不变属性，它们的值应该始终有效。例如，一个订单对象的数量必须始终为正整数，还要有名称和价格。因此，保证有效性是领域实体的责任（特别是聚合根）且一个无效的领域对象不应该存在。

实体:

- 包含领域业务逻辑。尽量不要在服务中包含业务逻辑，这会导致 [贫血领域模型(Anemic Domain Model)](https://martinfowler.com/bliki/AnemicDomainModel.html) （作为例外情况，我们可以把那些无法放到单个领域实体中的业务逻辑放到领域服务）。
- 给实体一个身份表示以把它和其他实体区分开来。这个身份标识在它的生命周期中是不变的。
- 两个实体的相等性是通过比较他们的身份标识符（一般就是它们的id字段）来确定的。
- 可以包含其他对象，比如其他实体或值对象。
- 了解自己的状态如何变更。
- 负责协调对起所拥有的对象的操作。
- 对上层无感知（服务，控制器等）。
- 领域实体数据应该被建模以适应业务逻辑，而不是数据库模式。
- 实体应该保护它的有效性，尽量避免使用public setters - 尽量使用方法来更新状态并在需要时在每次更新时执行有效性验证（可以用一个简单的`validate()`方法来检查业务逻辑没有被更新所破坏）。
- 必须在创建时保证一致性。在创建时验证实体和其他领域对象，在遇到错误时第一时间抛出异常。 [快速失败(Fail Fast)](https://en.wikipedia.org/wiki/Fail-fast).
- 避免使用无参构造函数，可以在构造函数（或者一个类似`create()`的工厂方法）中接受并验证所有必须的属性。
- 对于可选属性需要一些复杂的设置过程，可以使用 [Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) and [建造者模式(Builder Pattern)](https://refactoring.guru/design-patterns/builder)。
- 使实体部分不可变。确定哪些属性在创建后不能被更改，并使它们是`只读`（例如`id` 或 `createdAt`）的。

**注意**：很多人倾向于为每个实体创建一个模块，这种方式并不是很好。每个模块可能有多个实体。请记住一件事，将多个实体放在同一个模块中的条件是它们在业务逻辑上有关联，请勿把无关的实体放在同一个模块中。

示例文件：

- [user.entity.ts](src/modules/user/domain/entities/user.entity.ts)

阅读更多：

- [Domain Entity pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-entity-pattern.html)
- [Secure by design: Chapter 6 Ensuring integrity of state](https://livebook.manning.com/book/secure-by-design/chapter-6/)

---

## 聚合(Aggregates)

[聚合(Aggregate)](https://martinfowler.com/bliki/DDD_Aggregate.html) 是可以被以视为单个单元的领域对象的簇。它封装了那些在概念上可以划为一类的实体和值对象。它还包含了一组能在这些对象上使用的操作。

- 聚合通过将多个领域对象封装在一个单一抽象中来帮助简化领域模型。
- 聚合不应受数据模型的影响。领域对象之间的关联和数据库关系不同。
- 聚合根是一个包含了其他实体/值对象和那些操作它们的所有逻辑的实体。
- 聚合根拥有全局标识符 ([UUID / GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) / 主键)。聚合边界内的实体拥有局部的标识符，仅在聚合内是唯一的。
- 聚合根是通向整个聚合的入口。来自聚合外部的任何的引用都应该**仅**到达聚合根。
- 对一个聚合的任何操作都必须是 [事务性操作(transactional operations)](https://en.wikipedia.org/wiki/Database_transaction)。要么保存/更新/删除所有内容，要么什么都不做。
- 只有聚合根可以直接通过数据库查询获得。其他的一切都必须通过遍历得到。
- 和`实体`类似，聚合必须在整个生命周期中保护它们的有效性。当对聚合边界内有任何对象进行更新时，必须满足整个聚合中的所有不变量。简单来说，聚合内的所有对象必须是一致的，这意味着如果聚合内的一个对象状态发生改变，这不应该导致和其他领域对象发生冲突（这也被称为 _一致性边界_）。
- 聚合内的对象可以通过全局唯一标识符(id)引用其他聚合根。请避免直接持有对象引用。
- 尽量避免过大的聚合，这可能导致性能和可维护性问题。
- 聚合可以发布`领域事件`（详见下文）。

所有这些规则都来自围绕聚合创建边界的想法。边界简化了业务模型，因为它迫使我们在一套定义良好的规则中非常仔细地考虑每种关系。

总之，如果你在一个根`实体`中将多个相关实体和值对象组合在一个根`实体`中，则这个根`实体`就变成了一个`聚合根`，而这些相关实体和值对象的簇就就变成一个`聚合`。

示例文件：

- [aggregate-root.base.ts](src/libs/ddd/domain/base-classes/aggregate-root.base.ts) - 抽象基类。
- [user.entity.ts](src/modules/user/domain/entities/user.entity.ts) - 聚合其实只是一个遵循上述一些列规则的实体而已。

阅读更多：

- [Understanding Aggregates in Domain-Driven Design](https://dzone.com/articles/domain-driven-design-aggregate)
- [What Are Aggregates In Domain-Driven Design?](https://www.jamesmichaelhickey.com/domain-driven-design-aggregates/)
- [Effective Aggregate Design Part I: Modeling a Single Aggregate](https://www.dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_1.pdf)
- [Effective Aggregate Design Part II: Making Aggregates Work Together](https://www.dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_2.pdf)

---

## 领域事件(Domain Events)

领域事件表示领域中发生了你希望领域中的其他部分意识到的某些事情。领域事件仅仅是在内存中被送给领域事件调度器的消息而已。

例如，如果一个用户购买了某物，你可能需要做如下操作：

- 更新他的购物车；
- 从他的钱包进行结算；
- 创建一个新的发货单；
- 执行与执行“购买”操作无关的其他领域操作。

通常使用的典型方法是在执行购买操作逻辑的服务中执行所有这些逻辑。但这会在不同子领域中制造耦合。

另一种方式是发布一个`领域事件`。如果执行一个与聚合实例相关的命令需要在多个附加聚合上执行附加域的规则，你可以设计和实现一个领域事件来触发这些副作用。通过订阅具体的`领域事件`并根据需要创建尽可能多的事件处理程序，我们可以在同一个领域模型内的多个聚合中传播状态更改。这可以防止在聚合之间制造耦合。

通过将每个领域事件保存至数据库，领域事件可以通过创建 [审计日志(audit log)](https://en.wikipedia.org/wiki/Audit_trail) 来跟踪对重要实体的所有变更。阅读更多关于审计日志为什么有用的信息：[为什么软删除是邪恶的以及该怎么做(Why soft deletes are evil and what to do instead)](https://jameshalsall.co.uk/posts/why-soft-deletes-are-evil-and-what-to-do-instead).

领域事件（或其他任何东西）在单个进程中跨多个聚合中的所作出的所有变更都应该保存在单个数据库事务中以维护一致性。将整个工作流程包装在一个事务中或者使用像 [工作单元(Unit of Work)](https://java-design-patterns.com/patterns/unit-of-work/) 模式或者类似方法有助于实现这一点。

There are multiple ways on implementing an event bus for Domain Events, for example by using ideas from patterns like [Mediator](https://refactoring.guru/design-patterns/mediator) or [Observer](https://refactoring.guru/design-patterns/observer).

示例：

- [domain-events.ts](src/libs/ddd/domain/domain-events/domain-events.ts) - 这个类负责为任何需要发出或监听事件的对象提供发布/订阅机制。请记住这只是个概念验证的示例，也许不是实际产品应用中的最佳解决方案。
- [user-created.domain-event.ts](src/modules/user/domain/events/user-created.domain-event.ts) - 持有与已发布事件相关的数据的简单对象。
- [create-wallet-when-user-is-created.domain-event-handler.ts](src/modules/wallet/application/event-handlers/create-wallet-when-user-is-created.domain-event-handler.ts) - 这是当某个动作被执行后（在这个例子中是在用户被创建的同时创建他的钱包）触发的领域事件的处理器的一个示例。
- [typeorm.repository.base.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm.repository.base.ts) - 当存储库将变更持久化到一个聚合后，它会发布所有领域事件以供事件处理器执行。
- [typeorm-unit-of-work.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm-unit-of-work.ts) 
- [typeorm-unit-of-work.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm-unit-of-work.ts) - 这确保了所有变更都会在一个单独的数据库事务中执行。请记住这是一个简单的工作单元实现，因为它只是将执行过程包装到一个事务中。正确的工作单元实现需要先将所有变更保存在内存中。[Mikro-orm](https://www.npmjs.com/package/mikro-orm) 是 node.js 中可以被用来替代 [typeorm](https://www.npmjs.com/package/typeorm) 的一个优秀的 ORM 实现，它实现了正确的工作单元模式。阅读更多关于 [mikro-orm unit of work](https://mikro-orm.io/docs/unit-of-work/) 的信息。
- [unit-of-work.ts](src/infrastructure/database/unit-of-work/unit-of-work.ts) - 在这里你可以为事务中使用的那些特定的领域存储库创建工厂。
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 在这里我们从`工作单元(UnitOfWork)`中获取用户存储库来执行一个事务。

想要进一步了解领域事件和它的实现可以阅读：

- [Domain Event pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/domain-event-pattern.html)
- [Domain events: design and implementation](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)

**补充**:

- 本项目使用自定义实现来发布领域事件。不使用 [Node Event Emitter](https://nodejs.org/api/events.html) 或其他提供事件总线的包（比如 [NestJS CQRS](https://docs.nestjs.com/recipes/cqrs)）是因为它们不提供 `await` 选项来等待所有事件结束，这在将所有事件封装成一个事务时很有用。

- 事务对一些操作并不是必须的（比如查询或其他一些不会对其他聚合造成副作用的操作），因此对于这些场景你可能可以不使用工作单元而仅通过在构造函数中注入一个常规的存储库而不是事务性的存储库。

- 但在复杂的具有很多步骤的工作流中仅使用事件也会造成难以跟踪在整个应用程序中发生的所有事情。一个事件可能会触发另一个事件，这个过程可能还会不断重复发生。为了跟踪整个工作流你不得不查看多个地方并搜索每个步骤中用到的事件处理器，这会导致代码难以维护。在这种情况下使用一个服务/编排器/中介者可能会比仅使用事件更好一些，因为此时你可以只在一个地方维护整个工作流。这可能会导致一些耦合，但能提高可维护性。别仅仅依赖事件，为你的项目选择合适的工具吧。

- 在某些情况下，你将无法将事件对多个聚合所做的所有更改保存到单个事务中。例如如果你正在使用跨越多个服务之间事务的微服务，或 [事件溯源模式(Event Sourcing pattern)](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)，每个聚合有一个流。这种情况下跨多个聚合保存的事件在最终可以是一致的（例如使用 [Sagas](https://microservices.io/patterns/data/saga.html) 配合补偿事件或者使用 [流程管理器(Process Manager)](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) 获取其他类似的东西）。

## 集成事件(Integration Events)

进程外通信（调用微服务，外部APIs）被称为`集成事件`。如果需要向外部进程发送领域事件，则领域事件处理器应发送`集成事件`。

集成事件通常应该在所有领域事件都被处理完且被所有变更都被保存到数据库之后发布。

为了在微服务里处理集成事件，你可能需要外部消息代理/事件总线例如 [RabbitMQ](https://www.rabbitmq.com/) 或 [Kafka](https://kafka.apache.org/) 配合 [Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)，[Change Data Capture](https://en.wikipedia.org/wiki/Change_data_capture), [Sagas](https://microservices.io/patterns/data/saga.html) 或一个 [流程管理器(Process Manager)](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) 来保证 [数据一致性(eventual consistency)](https://en.wikipedia.org/wiki/Eventual_consistency)。

阅读更多：

- [Domain Events vs. Integration Events in Domain-Driven Design and microservices architectures](https://devblogs.microsoft.com/cesardelatorre/domain-events-vs-integration-events-in-domain-driven-design-and-microservices-architectures/)

这里有一些关于分布式系统中的集成事件的可用模式：

- [Saga distributed transactions](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- [Saga vs. Process Manager](https://blog.devarchive.net/2015/11/saga-vs-process-manager.html)
- [The Outbox Pattern](https://www.kamilgrzybek.com/design/the-outbox-pattern/)
- [Event Sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)

---

## 领域服务(Domain Services)

Eric Evans 在领域驱动设计中写道:

> 领域服务用于“领域中不属于实体或值对象的那些天然重要过程或转换”。

- 领域服务是特殊的领域层，用于执行依赖于两个或更多`实体`的领域逻辑。
- 当将逻辑放在`实体`中会破坏它的封装性且需要`实体`了解它本不应该了解的东西时，我们会将这些逻辑放在领域服务中。
- 相对于应用程序服务，领域服务是非常细粒度的，因为应用程序服务作为门面在外层提供一个API。
- 领域服务只操作属于本领域的类型。它们包含通用语言中的有意义的概念。它们持有那些不适合放在值对象和实体中的操作。

---

## 值对象(Value Objects)

一些属性和行为可以被从实体本身移出并放到`值对象`中。

值对象：

- 没有身份标识符。通过结构属性来判定相等性。
- 不可变。
- 可以当做`实体`或其他`值对象`的属性。
- 明确定义并强制执行重要的约束（不变量）。

值对象不应仅仅是一个方便的属性分组，而应该在领域模型中形成一个明确定义的概念。即使它只包含一个属性也是如此。当做为一个概念整体被建模时，它在传递时具有意义，而且它可以维护自身的约束。

`值对象`不仅是一个持有值的的数据结构。它还可以封装与其所代表的概念相关的逻辑。

示例文件：

- [address.value-object.ts](src/modules/user/domain/value-objects/address.value-object.ts)

阅读更多关于值对象的信息：

- [Martin Fowler blog](https://martinfowler.com/bliki/ValueObject.html)
- [Value Objects to the rescue](https://medium.com/swlh/value-objects-to-the-rescue-28c563ad97c6).
- [Value Object pattern](https://badia-kharroubi.gitbooks.io/microservices-architecture/content/patterns/tactical-patterns/value-object-pattern.html)

## 强制领域对象的有效性(Enforcing invariants of Domain Objects)

### 用值对象替换原始值

大部分代码库都对原始类型进行操作 - `字符串`，`数字` 等。在领域模型中，这种级别的抽象可能太低了。

可以使用特定类型和类来表达重要的业务概念。`值对象` 可以被用来替代原始类型以避免[primitives obsession](https://refactoring.guru/smells/primitive-obsession)
因此，例如，`string` 类型的 `email` ：

```typescript
email: string;
```

可以使用 `值对象` 来表示：

```typescript
email: Email;
```

现在创建一个 `email` 的唯一方式就是先创建一个新的 `Email` 类实例， 这样可以保证在创建时可以验证参数以防止错误值进入`实体`。

另外领域原始值的重要行为也可以被封装在一个地方。通过让领域原始值拥有并控制领域操作，你可以降低由于缺乏操作所涉及概念的详细领域知识而导致的错误风险。

为原始值创建对象可能有点麻烦，但它在某种程度上强制一个开发者更详细地研究领域，而不是仅仅使用一个原始类型，甚至不考虑该值在领域中代表什么。

为原始类型使用 `值类型` 也被称为 `领域原始类型`。这个概念和命名是 ["Secure by Design"](https://www.manning.com/books/secure-by-design) 这本书中建议的。

使用 `值类型` 替代原始类型可以：

- 通过使用 [通用语言(ubiquitous language)](https://martinfowler.com/bliki/UbiquitousLanguage.html) ，而不仅仅是`字符串`，来使代码更易理解。
- 通过确保每个属性的有效性来提高安全性。
- 封装与每个值有关的特定的业务规则。

作为一个地带方案，还可以使用 [类型别名(type alias)](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases) 来创建对象，这种方法仅仅是给原始值赋予一个语义上的名称。

**注意**：不要在数据传输对象(DTOs)中包含值对象、命令、事件、数据库模型、映射等。请先将它们转换成原始类型。值对象应该只在同一个边界上下文中使用。将它们发送到其他上下文中如命令/事件总线、保存至数据库等是一种不好的做法。因此这制造了耦合。

示例文件：

- [email.value-object.ts](src/modules/user/domain/value-objects/email.value-object.ts)

推荐阅读：

- [Primitive Obsession — A Code Smell that Hurts People the Most](https://medium.com/the-sixt-india-blog/primitive-obsession-code-smell-that-hurt-people-the-most-5cbdd70496e9)
- [Value Objects Like a Pro](https://medium.com/@nicolopigna/value-objects-like-a-pro-f1bfc1548c72)
- [Developing the ubiquitous language](https://medium.com/@felipefreitasbatista/developing-the-ubiquitous-language-1382b720bb8c)

**使用值对象/领域原始值和类型系统使程序中的非法状态不可表示**

有些人推荐把**所有**值都设计成对象：

引用自 [John A De Goes](https://twitter.com/jdegoes):

> 使非法状态不可表示也就是静态地证明运行时值（在没有异常的情况下）即相当于业务领域中的有效对象。这种技术在消除无意义的运行时状态方面的效果是惊人的，怎么强调都不为过。

让我们区分两种杜绝非法状态的保护措施：在**编译期**和在**运行时**。

### 在编译期

类型为开发者提供了有用的语义信息。好的代码应该要易于被正确使用，难以被误用。类型系统对此助益良多。它可以在编译期防止一些严重的错误，因此IDE可以立即显示类型错误。

最简单的例子可能是使用枚举替代常量，并用这些枚举作为某些东西的输入类型。当传入任何非预期的值时IDE会显示一个类型错误。

或者比如，假设一个业务逻辑需要一个人的联系方式，可以是 `email` 或 `phone`，或者两者都有。`email` 和 `phone` 都可以被标记为可选，例如：

```typescript
interface ContactInfo {
  email?: Email;
  phone?: Phone;
}
```

但如果开发者没有提供任何值会发生什么？业务规则被破坏。出现了非法状态。

解决方案：可以使用 [连接类型(union type)](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#union-types) 来表示这种情况。

```typescript
type ContactInfo = Email | Phone | [Email, Phone];
```

现在要么必须提供 `Email` 或 `Phone`， 要么二者都提供。如果什么值都不提供IDE将立即显示一个类型错误。现在业务规则验证从运行时被移动到了**编译期**，这将使应用程序更安全且能能够在一些非预期的事情发生时快速给出反馈。

这被称为 _类型状态模式_。

> 类型状态模式(ypestate pattern)是一种API设计模式，它在其编译时类型中编码有关对象运行时状态的信息。

阅读更多关于类型状态模式的信息：

- [Typestates Would Have Saved the Roman Republic](https://blog.yoavlavi.com/state-machines-would-have-saved-the-roman-republic/)
- [The Typestate Pattern](https://cliffle.com/blog/rust-typestate/)

### 在运行时

那些无法在编译期验证的东西（比如用户输入）会在运行时被验证。

领域对象必须保护它们的合法性。使用一些验证规则来保护它们的状态免受损坏。

`值对象`可以表示成领域中的类型化值。这里的目标是仅封装与表示字段相关的验证和业务逻辑，并通过首先强制创建有效的`值对象`来禁止传递原始值。该对象只接受在其上下文中有意义的值。

如果方法的每个参数和返回值根据定义都是有效的，你将在你的代码库中的所有单个方法调用中不费吹灰之力地验证输入和输出参数。这将使应用程序在遇到错误时更具弹性，同时还将白保护它免受由无效输入数据带来的安全漏洞的影响。

数据不应该被信任。很多情况下，无效数据可能会出现在领域中。例如，如果数据来组外部API，数据库或程序错误导致的错误数据。

强制自我验证将在数据损坏时立即通知你。不验证领域对象会允许他们处于一个非法状态，这会导致错误。

> 如果没有领域原始值，其余代码需要处理验证，格式化，比较和很多其他细节。实体代表长时间存在的具有唯一标识符对象，例如新闻摘要中的文章，宾馆里的房间和线上购物中的购物车。系统中的功能通常以改变这些对象的状态为中心：宾馆房间被预定，购物车中的商品被支付，诸如此类。迟早控制流将被引导到代表这些实体的代码。如果所有数据都被作为通用类型传输，例如整型或字符串，则责任就落到了实体代码上，以验证、比较和格式化数据，以及其他任务。此时实体代码将需要负担很多此类任务而不是专注于它所建模的业务流状态变化。使用领域原始值可以抵消实体变得日益复杂的趋势。

引用自 [Secure by design: Chapter 5.3 Standing on the shoulders of domain primitives](https://livebook.manning.com/book/secure-by-design/chapter-5/96)

**注意**：尽管 _primitive obsession_ 是一个代码坏味道，但一些人认为为每个原始值都创建一个类/对象可能是一种过度设计。对于不太复杂的和小型项目这绝对是一种过度设计。对于较大的项目，这个方法有人支持也有人反对。如果不希望为每个原始值创建一个类，那就为那些拥有特定规则和行为的原始值创建类，或者仅仅使用某些验证框架在领域外部进行验证。关于这个话题这里有一些思考：[From Primitive Obsession to Domain Modelling - Over-engineering?](https://blog.ploeh.dk/2015/01/19/from-primitive-obsession-to-domain-modelling/#7172fd9ca69c467e8123a20f43ea76c2)。

**推荐阅读**：

- [Making illegal states unrepresentable](https://v5.chriskrycho.com/journal/making-illegal-states-unrepresentable-in-ts/)
- [Domain Primitives: what they are and how you can use them to make more secure software](https://freecontent.manning.com/domain-primitives-what-they-are-and-how-you-can-use-them-to-make-more-secure-software/)
- ["Secure by Design" Chapter 5: Domain Primitives](https://livebook.manning.com/book/secure-by-design/chapter-5/) (a full chapter of the article above)

### 保护和验证(Guarding vs validating)

你可能已经注意到了，我们在两个地方做验证：

1. 第一次是当用户输入发送到我们的应用程序时。在我们的示例中我们使用数据传输对象(DTO)装饰器：[create-user.request-dto.ts](src/modules/user/commands/create-user/create-user.request.dto.ts)。
2. 第二次是在领域对象中，例如： [email.value-object.ts](src/modules/user/domain/value-objects/email.value-object.ts).

那么，为什么要验证两次？让我们把第二次验证称为 "_保护_"，保护和验证的区别如下：

- 保护是一种故障安全机制。领域层将其视为保证领域模型始终有效的的数据有效性机制。
- 验证是一种过滤机制。外层将它们视为输入验证规则。

> 这种差异导致对这两种违反业务规则的情况的不同处理。违反领域模型中的数据有效性是一种异常情况，需要抛出异常。另一方面，外部输入正确不属于异常，

在将来自外部世界的输入传递到领域模型之前需要对它们进行过滤。这是防止数据不一致的第一道防线。在这个阶段，任何不正确的数据都会被拒绝并提供相应的错误消息。
一旦过滤确认了传入的数据有效，它就会被传入领域中。当数据到达始终有效的领域边界时，它会被假设是有效的，此时任何违反数据有效性的行为意味着你引入了一个bug。
保护有助于揭示这些错误。它们是故障安全机制，是确保始终有效边界中的数据确实有效的最后一道防线。和验证不同的是，保护会抛出异常；保护遵循 [快速失败原则(Fail Fast principle)](https://enterprisecraftsmanship.com/posts/fail-fast-principle)。

领域类应该始终确保自身是有效的。

为了防止 null/undefined 值，空对象和空数组，不正确的输入长度等。可以创建一个库 [guards](<https://en.wikipedia.org/wiki/Guard_(computer_science)>) 来专门处理这类验证。

示例文件：[guard.ts](src/libs/ddd/domain/guard.ts)

**请注意** 并非所有验证/保护都可以在单个领域对象中完成，它应该只验证所有上下文共享的规则。在某些情况下，验证会因所依赖的上下文而异，或者一个字段的验证可能涉及另一个字段，又或者是一个不同的实体。请根据实际情况处理。

阅读更多：

- [Refactoring: Guard Clauses](https://medium.com/better-programming/refactoring-guard-clauses-2ceeaa1a9da)
- [Always-Valid Domain Model](https://enterprisecraftsmanship.com/posts/always-valid-domain-model/)

<details>
<summary><b>注意</b>: 使用验证库而不是自定义的 _guards_</summary>

您可以使用外部验证库代替自定义 _guards_，但将领域绑定到外部库并不是一个好习惯，通常不建议这样做。

在有需要时可以特殊处理，尤其是对于仅验证一件事（像是特定 ID，例如比特币钱包地址）的那些非常具体的验证库。将一个或几个`值对象`绑定到这样一个特定的库并不会有什么危害。与通用验证库不同，后者将在在任何地方和领域相关联，并且在旧库不再维护、包含严重bug或被黑客利用等情况下，在每个`值对象`中更新它会很麻烦。

尽管如此，在领域**外部**使用验证框架进行完整性验证还是可行的 (例如 [class-validator](https://www.npmjs.com/package/class-validator) 在 `DTOs` 上使用装饰器)，并且在领域内部除了业务规则之外只做一些基本检查（保护），例如检查 `null` 或 `undefined`，检查数据长度，匹配简单的正则表达式等来检查传入的值是否有意义以提高安全性。

<details>
<summary>使用正则表达式的注意事项</summary>

在使用自定义正则表达式验证像 `email` 这种数据时要很小心，尽量只在验证非常简单的规则时使用自定义的正则表达式，如果你并不太擅长正则表达式，最好使用专门的验证库来处理那些复杂的验证规则，以免出错。

另外，请注意在某些验证类型上，自定义的正则表达式可能会与你在领域外使用的验证库的正则表达式相冲突。

举个例子，值可以被验证库验证为有效，但 `值对象` 可能会由于自定义正则表达式不健全而抛出错误（正确验证 `email` 的正则表达式可比随便从google上复制下来的正则表达式复杂多了。尽管如此，还是可以通过一个简单的规则进行验证，该规则始终为真并且不会引起任何冲突，例如每个 `email` 必须包含一个`@`）。请尝试查找和验证不会导致冲突的模式。

---

</details>

虽然还有其他关于如何在领域内进行验证的策略，比如在创建新的 `值对象` 时将验证模式作为依赖项传入，但这会导致额外的复杂性。

是否在领域内使用外部库/框架进行验证是一种权衡取舍，请分析所有的利弊，选择更适合当前应用程序的方法。

对于某些项目，尤其是较小的项目，仅使用一个验证库/框架来做验证可能更简单也更合适。

</details>

### 验证的类型

这里有一些推荐的验证顺序。推荐把像是检查 null/undefined 和检查数据长度这类低开销的操作放在最前面，那些高开销的例如查询数据库这类操作放在后面。

最好按照以下顺序做验证：

- _数据源 - 是佛偶来自合法的发送方？_ 只要有可能，根据实际情况只接受来自已授权用户/IP白名单等的数据。
- _Existence - are provided data not empty?_ Further validations make no sense if data is empty. Check for empty values: null/undefined, empty objects and arrays.
- _存在性 - 提供的数据不为空吗？_ 当数据为空时，进一步的验证没有意义。请对空值进行检查：null/undefined, 空对象和空数组。
- _数据大小 - 是否过大？_ 再进一步往下验证前，不管数据是什么类型的，先检查输入数据的长度/大小。这可以防止因数据过大导致整个进程阻塞（发送过大的数据可能是一个 [DoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) 攻击）。
- _词法内容 - 是否包含正确的字符和编码？_ 例如，如果我们期望数据只包含数字，我们就扫描它以检查是否包含非数字的内容。如果发现它包含非期望的内容，我们就可以认定数据要么由于某种错误而损坏，要么是被恶意制作的以欺骗我们的系统。
- _语法 - 格式正确吗？_ 检查数据以确保其格式正确。有时语法检查就像使用正则表达式一样简单，但也可能更复杂，比如解析一个 XML 或 JOSN。
- _语义 - 数据有意义吗？_ 使用系统的其余部分（例如数据库，或其他进程）检查数据。例如，在数据库中检查一个ID是否存在。

阅读更多关于验证的类型相关的信息：

- ["Secure by Design" Chapter 4.3: Validation](https://livebook.manning.com/book/secure-by-design/chapter-4/109).

## 领域错误(Domain Errors)
