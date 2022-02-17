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
  - [应用程序核心(Application Core)](#应用程序核心(Application-Core))
  - [应用层(Application Layer)](#应用层(Application-Layer))
    - [应用服务(Application Services)](#应用服务(Application-Services))
    - [命令和查询(Commands and Queries)](#命令和查询(Commands-and-Queries))
    - [端口(Ports)](#端口(Ports))
  - [领域层(Domain Layer)](#领域层(Domain-Layer))
    - [实体(Entities)](#实体(Entities))
    - [聚合(Aggregates)](#聚合(Aggregates))
    - [领域事件(Domain Events)](#领域事件(Domain-Events))
    - [集成事件(Integration Events)](#集成事件(Integration-Events))
    - [领域服务(Domain Services)](#领域服务(Domain-Services))
    - [值对象(Value Objects)](#值对象(Value-Objects))
    - [强制领域对象的有效性(Enforcing invariants of Domain Objects)](#强制领域对象的有效性(Enforcing-invariants-of-Domain-Objects))
    - [领域错误(Domain Errors)](#领域错误(Domain-Errors))
    - [在应用核心内部使用库(Using libraries inside application's core)](#在应用核心内部使用库(Using-libraries-inside-application's-core))
  - [接口适配器(Interface Adapters)](#接口适配器(Interface-Adapters))
    - [控制器(Controllers)](#控制器(Controllers))
      - [Resolvers](#resolvers)
    - [数据传输对象(DTOs)](#数据传输对象(DTOs))
      - [请求数据传输对象(Request DTOs)](#请求数据传输对象(Request-DTOs))
      - [响应数据传输对象(Response DTOs)](#响应数据传输对象(Response-DTOs))
      - [其他建议(Additional recommendations)](#其他建议(Additional-recommendations))
      - [本地数据传输对象(Local DTOs)](#本地数据传输对象(Local-DTOs))
  - [基础设施(Infrastructure)](#基础设施(Infrastructure))
    - [适配器(Adapters)](#适配器(Adapters))
    - [存储库(Repositories)](#存储库(Repositories))
    - [持久化模型(Persistence models)](#持久化模型(Persistence-models))
    - [其他可以作为基础设施层一部分的东西(Other things that can be a part of Infrastructure layer)](#其他可以作为基础设施层一部分的东西(Other-things-that-can-be-a-part-of-Infrastructure-layer))

- [其他建议(Other recommendations)](#其他建议(Other-recommendations))
  - [对小型APIs的建议(Recommendations for smaller APIs)](#对小型APIs的建议(Recommendations-for-smaller-APIs))
  - [架构、最佳实践、设计模式和原则的通用建议(General recommendations on architectures, best practices, design patterns and principles)](#架构、最佳实践、设计模式和原则的通用建议(General-recommendations-on-architectures,-best-practices,-design-patterns-and-principles))
  - [行为测试(Behavioral Testing)](#行为测试(Behavioral-Testing))
  - [目录和文件结构(Folder and File Structure)](#目录和文件结构(Folder-and-File-Structure))
    - [文件名(File names)](#文件名(File-names))
  - [自定义实用类型(Custom utility types)](#自定义实用类型(Custom-utility-types))
  - [避免大规模继承链(Prevent massive inheritance chains)](#避免大规模继承链(Prevent-massive-inheritance-chains))
- [其他资源(Additional resources)](#其他资源(Additional-resources))
  - [文章(Articles)](#文章(Articles))
  - [Github仓库(Github Repositories)](#Github仓库(Github-Repositories))
  - [文档站点(Documentation Websites)](#文档站点(Documentation-Websites))
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

# 应用层(Application Layer)

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

应用程序核心和领域层不应该抛出 HTTP 异常或状态因为它不应该了解自己在哪个上下文中被使用，因为它可以被任何东西使用：HTTP 控制器、微服务事件处理器，命令行接口等。更好的做法是使用适当的错误代码创建自定义错误类。

异常是针对特殊情况的。复杂的领域通常拥有很错误，这些错误并非异常，而是业务逻辑的一部分（例如“座位已被预订，请选择其他座位”）。这些错误可能需要特殊处理。这种情况下显式返回错误类型会比直接抛出异常更合适。

返回错误而不是显式地抛出异常使你可以知晓一个方法能够返回的每个异常的类型，以便你可以根据情况进行处理。这可以使错误处理和跟踪更简单。

为了帮助解决这个问题，你可以使用某种 Result 对象中的 Success 或 Failure (一种借鉴自函数式编程语言，比如 Haskell 的 `Either` [monad](<https://en.wikipedia.org/wiki/Monad_(functional_programming)>))。和抛出异常不同的是，这种方法允许我们对每种错误定义类型，并强制你显式地处理成功和失败的情况而不是使用 `try/catch`。例如：

```typescript
if (await userRepo.exists(command.email)) {
  return Result.err(new UserAlreadyExistsError()); // <- returning an Error
}
// else
const user = await this.userRepo.create(user);
return Result.ok(user);
```

[@badrap/result](https://www.npmjs.com/package/@badrap/result) - 如果你想使用 Result 对象，这里有一个不错的 npm 包。

返回错误而不是抛出异常会增加一些样板代码，但它可以使你的应用程序更健壮和安全。

**注意**：请区别领域错误和异常。异常通常会被抛出且没有返回值。如果你返回技术类异常（例如连接失败，进程内存不足等），可能会导致一些安全问题并违反 [快速失败(Fail-fast)](https://en.wikipedia.org/wiki/Fail-fast) 原则。返回一个异常而不是终止程序执行流将允许程序在一个非法状态下继续运行，这可能会导致发生更多的非预期错误，因此在这种情况下最好是抛出异常而不是返回它。

示例文件：

- [user.errors.ts](src/modules/user/errors/user.errors.ts) - 用户错误
- [create-user.service.ts](src/modules/user/commands/create-user/create-user.service.ts) - 注意如何返回 `Result.err(new UserAlreadyExistsError())` 错误而不是抛出异常。
- [create-user.http.controller.ts](src/modules/user/commands/create-user/create-user.http.controller.ts) 在 user http 控制器中我们包装一个错误并决定如何处理它。如果错误是 `UserAlreadyExistsError` 就抛出一个 `Conflict Exception` 异常，用户会收到 `409 - Conflict` 响应。如果是未知错误我们就抛出它，NestJS 会返回 `500 - Internal Server Error` 给用户。
- [create-user.cli.controller.ts](src/modules/user/commands/create-user/create-user.cli.controller.ts) - 在 CLI 控制其中我们不需要返回正确的错误码，因此我们只需要 `.unwrap()` 一个结果，它只会在出现错误时抛出异常。
- [exceptions](src/libs/exceptions) 这个目录包含了一些通用的应用程序异常（非领域特定的）。
- [exception.interceptor.ts](src/infrastructure/interceptors/exception.interceptor.ts) - 在这个文件中我们将应用通用异常转换成NestJS HTTP 异常。这样我们就不受框架或 HTTP 协议的束缚。

阅读更多：

- [Flexible Error Handling w/ the Result Class](https://khalilstemmler.com/articles/enterprise-typescript-nodejs/handling-errors-result-class/)
- [Advanced error handling techniques](https://enterprisecraftsmanship.com/posts/advanced-error-handling-techniques/)
- ["Secure by Design" Chapter 9.2: Handling failures without exceptions](https://livebook.manning.com/book/secure-by-design/chapter-9/51)

## 在应用核心内部使用库(Using libraries inside application's core)

是否在应用程序核心中尤其是领域层使用库是一个充满争议的话题。在现实世界中，注入每个库而不是直接导入它这种做法并不实用，因此可以为一些有助于实现领域逻辑的单一职责的库（例如处理数字的库）做一些例外处理。

需要谨记的是导入到应用程序核心中的库**不应该**暴露：

- 能够访问任何进程外部资源（HTTP请求，数据库访问等）的功能；
- 与领域无关的功能（框架，技术细节例如ORMs，Logger等）。
- 那些带来随机性的功能（生成随机IDs，时间戳等）因为这会使测试结果不可预期（尽管在 TypeScript 世界中这没什么大不了的，因为这可以在不使用 依赖注入 DI 的情况下被测试库模拟）；
- 如果一个库经常变更或者它本身有很多依赖，那么它很可能不应该被用于领域层。

考虑使用 [适配器(adapter)](https://refactoring.guru/design-patterns/adapter) 或 [门面(facade)](https://refactoring.guru/design-patterns/facade) 模式赖创建一个 `防腐` 层以使用这些库。

我们有时会容忍那些集中使用的库，但请小心那些可能分布在许多领域对象中的通用库。如果需要，替换这些库将很困难。只将一个或少数几个领域对象绑定到一些单一职责库上是没问题的。替换只与一个或少数几个对象绑定的库要比替换那些无处不在的通用库要容易得多。

除了各式各样的库以外，还有框架。框架可能是一个真正的累赘，因为根据定义框架处于一个支配者的地位，并且当你的应用程序与框架粘合在一起时你将很难替换它。在外层（例如基础设施层）中使用框架是个不错的选择，但请记得尽量保持你的领域是干净的。你应该能够提取你的领域层并使用任意其他框架围绕它构筑一个新的基础设施而不会破坏它的业务逻辑。

NestJS 在这方面做得很好，因为它使用了低侵入性的装饰器，因此你可以使用像 `@Inject()` 这种装饰器，而不影响你的业务逻辑，并且在需要时移除或替换它也相对容易。无须完全放弃框架，但请把它们控制在边界之内以防止它们影响你的业务逻辑。

尽可能多地不要让核心承担无关的职责，特别是领域层。此外，尽量减少通用依赖的使用。你的软件中的更多依赖意味着更多潜在错误和安全漏洞。一个使你的软件更健壮的技术手段是减少你的软件的依赖 - 依赖越少，则出错的可能性越小。另一方面，移除所有依赖将会适得其反，自己造轮子的工作量巨大且与那些被广泛使用的依赖库相比可靠性更低。重要的是找到一个好的平衡点，这需要经验。

阅读更多：

- [Referencing external libs](https://khorikov.org/posts/2019-08-07-referencing-external-libs/).
- [Anti-corruption Layer — An effective Shield](https://medium.com/@malotor/anticorruption-layer-a-effective-shield-caa4d5ba548c)

---

# 接口适配器(Interface Adapters)

接口适配器（也被称为驱动/主适配器）是面向用户的接口，它从用户那里获取输入数据并将其重新打包成一种便于用例（服务/命令处理器）和实体使用的形式。接着它们从这些用例和实体那里获取输出，再从新打包成便于向用户展示的形式。这里的用户可以是一个使用应用程序的人也可以是一台服务器。

包含 `控制器` 和 `请求`/`响应` DTOs （如果需要，还可以包含 `Views`，例如服务端生成的 HTML 模板）。
 
## 控制器(Controllers)

- 控制器是一个面向用户的API，用于解析请求，触发业务逻辑以及将结果呈现给客户端。
- 每个用例一个控制器被认为是一种好的实践。
- 在 [NestJS](https://docs.nestjs.com/) 的世界中，控制器可能是一个使用 OpenAPI/Swagger decorators](https://docs.nestjs.com/openapi/operations) 来文档化 API 的好地方。

可以使用每个触发器类型一个控制器来实现更清晰地分离。例如：

- [create-user.http.controller.ts](src/modules/user/commands/create-user/create-user.http.controller.ts) 用于 HTTP 请求 ([NestJS Controllers](https://docs.nestjs.com/controllers))，
- [create-user.cli.controller.ts](src/modules/user/commands/create-user/create-user.cli.controller.ts) 用于命令行访问 ([NestJS Console](https://www.npmjs.com/package/nestjs-console))
- [create-user.message.controller.ts](src/modules/user/commands/create-user/create-user.message.controller.ts) 用于外部消息 ([NetJS Microservices](https://docs.nestjs.com/microservices/basics))。
- 等等。

### Resolvers

如果你正在使用 [GraphQL](https://graphql.org/) 而不是控制器，你将会使用 [Resolvers](https://docs.nestjs.com/graphql/resolvers)。

分层架构的好处一直就是关注点分离。正如你所见，使用 [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) 或者 GraphQL 无关紧要，唯一发生改变的是面向用户的 API 层（接口适配器）。所有的应用程序核心都将保持不变，因为它不依赖于你所使用的技术。

示例文件：

- [create-user.graphql-resolver.ts](src/modules/user/commands/create-user/create-user.graphql-resolver.ts)

---

## 数据传输对象(DTOs)

来自外部应用的数据应该由特殊类型的类表示 - 数据传输对象 (简称[DTO](https://en.wikipedia.org/wiki/Data_transfer_object))。
数据传输对象是一种在进程间携带数据的对象。它定义了你的 API 和客户端之间的契约。

### 请求数据传输对象(Request DTOs)

用户发送的输入的数据。

- 使用请求数据传输对象提供了一个契约，你的 API 客户端必须遵循该约定才能发出正确的请求。

示例：

- [create-user.request.dto.ts](src/modules/user/commands/create-user/create-user.request.dto.ts)
- [create.user.interface.ts](src/interface-adapters/interfaces/user/create.user.interface.ts)

### 响应数据传输对象(Response DTOs)

返回给用户的输出数据。

- 使用响应数据传输对象保证了客户端只能收到 DTO 契约中描述的数据，而不是你的模型/实体所拥有的所有数据（这可能导致数据泄露）。

示例：

- [user.response.dto.ts](src/modules/user/dtos/user.response.dto.ts)
- [user.interface.ts](src/interface-adapters/interfaces/user/user.interface.ts)

---

使用 DTOs 可以保护你的客户端免受 API 内部数据结构变化的影响。当内部数据模型变化时（例如重命名变量或者拆分表），它们仍然可以被映射以匹配相应的 DTO 来对使用你的 API 的任何人保持兼容性。

当更新 DTO 接口时，可以通过在端点前加上版本号来创建一个新版本的 API，例如：`v2/users`。这将通过防止破坏那些使用你的 API 的更新缓慢的应用程序的兼容性而使升级变得轻松。

你可能已经注意到了 [create-user.command.ts](src/modules/user/commands/create-user/create-user.command.ts) 包含了和 [create-user.request.dto.ts](src/modules/user/commands/create-user/create-user.request.dto.ts) 相同的属性。
那么为什么我们在已经拥有携带属性的 Command 对象的情况下还需要 DTOs 呢？我们能否只留下其中一个以避免重复？

> 因为命令和 DTOs 是不同的事物，它们处理不同的问题。命令是可序列化的方法调用 - 调用领域模型的方法。然而 DTOs 是数据契约。引入这个带有数据契约的独立层是为 API 客户端提供向后兼容性。如果没有 DTOs，API 将会随着领域模型中的每个修改而被发生重大变化。

阅读更多关于这个主题的内容：[Are CQRS commands part of the domain model?](https://enterprisecraftsmanship.com/posts/cqrs-commands-part-domain-model/) （阅读  "_Commands vs DTOs_" 章节）。

### 其他建议(Additional recommendations)

- DTOs 应该是面向数据的，而不是面向对象的。它的属性应该主要是基本类型的。我们不在此做任何建模工作，只是发送原始数据。
- 在返回一个`响应`时，使用_白名单_过滤属性要优于使用_黑名单_。这确保了如果程序员忘记将不应返回给用户的新添加属性列入黑名单时不会泄漏敏感数据。
- `请求/响应`对象的接口应该被存放在某个共享目录中而不是放在模块目录中，因为它们可能会被用于不同的应用程序中（例如前端页面，手机端应用或微服务）。可以考虑为共享接口创建 git submodule 或者独立的包。
- `请求/响应` DTO 类可能是一个使用数据验证和装饰器的好地方，例如 [class-validator](https://www.npmjs.com/package/class-validator) 和 [class-sanitizer](https://www.npmjs.com/package/class-sanitizer) （请确保首先收集所有的验证错误，然后才将它们返回给用户，这被成为 [通知模式(Notification pattern)](https://martinfowler.com/eaaDev/Notification.html)。Class-validator 默认是这样处理的）。
- `请求/响应` DTO 类可能还是一个使用 Swagger/OpenAPI 库装饰器的好地方 [NestJS provides](https://docs.nestjs.com/openapi/types-and-parameters)。
- 如果不使用用于数据验证和文档的 DTO 装饰器，则 DTO 可以只是一个接口而不是类 + 接口。
- 如果使用 DTO 类，可以使用单独的映射器或在构造函数中将数据转换为 DTO 的格式。

### 本地数据传输对象(Local DTOs)

在某些项目中还可以看到一种叫本地 DTOs 的东西。有些人倾向于在领域之外（例如在控制器中）不使用领域对象（例如实体），而是返回一个普通的 DTO 对象。这种项目由于不使用像接口和数据映射这类技术而避免了额外的复杂性和样板代码。

[这里](https://martinfowler.com/bliki/LocalDTO.html) 是 Martin Fowler 关于本地 DTOs 的一些思考，简短摘录一些：

> 有些人主张将 DTOs 看作是服务层 API 的一部分，因为它们确保了服务层客户端不依赖于底层领域模型。虽然这可能很方便，但我认为这类数据映射没有必要。

当你需要对模块做解耦时可能回想引入本地 DTOs。例如，当在一个模块中查询另一个模块时，你并不想在模块间泄露你的实体。在这种情况下使用本地 DTO 可能是个更好的主意。

---

# 基础设施(Infrastructure)

基础设施负责维护技术细节。你可以在那里找到业务实体的数据库实现、消息代理、I/O组件、依赖注入、框架和任何其他代表架构细节的东西，主要是框架依赖，外部依赖等。

它是最不稳定的层。由于这一层中的事物很可能发生变化，我们让它们尽可能远离更稳定的领域层。因为它们被隔离在领域层之外，因此进行更改和将一个组件替换为另一个组件相对容易。

基础设施层可以包含 `适配器`，数据库相关文件例如 `存储库`、`ORM 实体`/`模式`、框架相关文件等。

## 适配器(Adapters)

- 基础设施层（也被叫作驱动/辅助适配器）允许一个软件系统通过在请求（例如持久化、消息代理、发送Email或消息、请求第三方 API 等）时接收、存储和提供数据来与外部系统交互。
- 适配器也可以被用来与单个进程中的不同领域交互以避免这些领域之间的耦合。
- 适配器本质上是端口的实现。它们不应该在代码中的任何位置被直接调用，只能通过端口（接口）调用。
- 适配器可以被用来作为旧代码的防腐层(ACL)。

阅读更多 ACL 的信息：[防腐层: 如何防止旧代码破坏新系统](https://www.cloudbees.com/blog/anti-corruption-layer-how-keep-legacy-support-breaking-new-systems)


适配器应该具有：

- 一个位于应用/领域层的它所实现的`端口`。
- 一个在数据和领域之间映射的映射器（如果有需要的话）。
- 一个用于接收数据的 DTO/接口。
- 一个验证器，以确保传入的数据没有损坏（验证可以在 DTO 类中使用装饰器，或者可以通过“值对象”进行验证）。

## 存储库(Repositories)

存储库是对存在于数据库中的实体集合的抽象。它们集中了通用数据访问功能并封装了访问这些数据所需的的逻辑。实体/聚合可以被存入存储库中并可以在之后被检索且领域无需知道这些数据被保存在何处：数据库中、文件中，或者是其他什么地方。

我们使用存储库将用于访问数据库的础设施或技术细节与领域模型层解耦。

Martin Fowler 这样描述存储库：

> 存储库执行领域模型层和数据映射之间的中介任务，起作用类似于内存中的一组领域对象。客户端对象以声明的方式构建查询并将它们发送给存储库以获取响应。从概念上来说，一个存储库封装了存储在数据库中的一组对象以及可以对它们执行的操作，提供了更接近持久化层的操作。存储库也支持在一个方向上清晰地分离工作领域和数据分配或映射的依赖关系。

这里的数据流看上去像这样：存储库从应用服务中收到一个领域`实体`，它将其映射成数据库的 schema/ORM 格式，执行必要的操作（保存/更新/检索等），然后将它映射会领域`实体`格式并返回给服务。

**请记住**应用程序核心不允许直接依赖于存储库，它应该依赖抽象(端口/接口)。这使得数据检索于技术无关。

示例文件：

本项目包含一个抽象存储库类以执行基本的 CRUD 操作：[typeorm.repository.base.ts](src/libs/ddd/infrastructure/database/base-classes/typeorm.repository.base.ts)。之后这个基础类将被一个具体的存储库类所扩展，实体需要的所有特定的操作都在这个特定的存储库类中实现，例如：[user.repository.ts](src/modules/user/database/user.repository.ts)。

## 持久化模型(Persistence models)

使用单个实体处理领域逻辑和数据库相关事务会导致以数据库为中心的架构。在 DDD 的世界里领域模型和持久化模型应该是分离的。

由于领域`实体`对其数据进行了建模，以便更好地适应领域逻辑，因此它可能不是保存在数据库中的最佳形式。为此目的，可以创建在所使用的特定数据库中具有更好的表示形式的`持久化模型`。领域层不应该知道任何持久化模型的信息，并且它也不应该知道。

可以有多个模型针对不同目的进行优化，例如：

- 具有自己的模型的领域 - `实体`、`聚合`和`值对象`。 
- 具有自己的模型的持久化层 - ORM ([Object–relational mapping](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping)), schemas, 读/写模型（如果数据库被分成一个读库和一个写库）([CQRS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)) 等。

随着时间的推移，当数据量增加时，可能需要对数据库进行一些更改，例如重新设计表结构甚至是改变使用的数据库来提高性能活泼数据完整性。在没有明确区分`领域`模型和`持久化`模型的情况下，任何对数据库的变更都会导致你的领域`实体`或`聚合`发生变化。例如，在执行一个数据库[规范化(normalization)](https://en.wikipedia.org/wiki/Database_normalization)时，数据可以分布在多个表中而不是一个表，反之，对于[反规范化(denormalization)](https://en.wikipedia.org/wiki/Denormalization)亦然。这可能会迫使团队为此对领域层进行复杂的重构，这可能带来意想不到的bug和挑战。分离领域模型和持久化模型可以阻止上述事情发生。

**注意**：分离领域模型和持久化模型对于较小的应用可能会矫枉过正，请三思而后行。

示例文件：

- [user.orm-entity.ts](src/modules/user/database/user.orm-entity.ts) <- 使用 ORM 的持久化模型。
- [user.orm-mapper.ts](src/modules/user/database/user.orm-mapper.ts) <- 持久化模型还应该有一个相应的映射器在领域模型和持久化模型之间做转换。

ORM 的替代方式是原始查询或某种查询构建器(例如 [knex](https://www.npmjs.com/package/knex))。对于更大的项目，这可能是比对象-关系映射更好的方法，因为它提供了更大的灵活性和更好的性能。

阅读更多：

- [Stack Overflow question: DDD - Persistence Model and Domain Model](https://stackoverflow.com/questions/14024912/ddd-persistence-model-and-domain-model)
- [Just Stop It! The Domain Model Is Not The Persistence Model](https://blog.sapiensworks.com/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx)
- [Comparing SQL, query builders, and ORMs](https://www.prisma.io/dataguide/types/relational/comparing-sql-query-builders-and-orms)
- [Secure by Design: Chapter 6.2.2 ORM frameworks and no-arg constructors](https://livebook.manning.com/book/secure-by-design/chapter-6/40)

## 其他可以作为基础设施层一部分的东西(Other things that can be a part of Infrastructure layer)

- 框架相关文件；
- 应用程序日志实现；
- 基础设置相关的事件([Nest-event](https://www.npmjs.com/package/nest-event))；
- 定时 cron 作业或任务启动器([NestJS Schedule](https://docs.nestjs.com/techniques/task-scheduling));
- 其他技术相关文件。

---

# 其他建议(Other recommendations)

## 对小型APIs的建议(Recommendations for smaller APIs)

在业务逻辑较少的中小型项目中实现复杂的架构要小心。部分构建块/模式/原则可能适用，但其他的可能是过度工程。

例如：

- 将代码分成模块/层/用例，适用一些构建块如控制器/服务/实体，划分边界和适用依赖注入等。对于任何项目来说都是一个好主意。
- 但是诸如为每个原始类型创建一个对象、使用`值对象`来将业务逻辑分成更小的类、将`领域模型”`与`持久化模型`等分开的实践，在那些以数据为中心的且只有很少或几乎没有业务逻辑的项目中，可能只会使此类解决方案复杂化且增加额外的样板代码、数据映射和维护开销，而不会带来太多好处。

[DDD](https://en.wikipedia.org/wiki/Domain-driven_design) 和这里描述的其他实践主要是关于创建具有复杂业务逻辑的软件。但是对于更简单的应用程序来说，有什么更好的方法吗？

对那些业务逻辑不多的应用程序来说，需要考虑其他架构。最流行的可能是 [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)。_模型-视图-控制器_更适用于几乎没有什么业务逻辑的 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)应用程序，因为它专注于主要是作为数据库视图的软件。

## 架构、最佳实践、设计模式和原则的通用建议(General recommendations on architectures, best practices, design patterns and principles)

不同的项目很可能会有不同的需求。此类项目中的一些原则/模式可以以简化的形式实现，还有一些可以被跳过。遵循 [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)原则且不要过度工程。

有时复杂架构和原则例如 [SOLID](https://en.wikipedia.org/wiki/SOLID)可能和[YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) 以及 [KISS](https://en.wikipedia.org/wiki/KISS_principle)并不兼容。一个优秀的程序员应该是务实的，并且必须能够将他的技能和知识与常识相结合，为问题选择最佳解决方案。

> 在这些建议能为你所用之前，你需要在实际项目中进行面向对象软件开发方面的一些经验。此外，这些建议并不会在你找到好的解决方案和走得太远提示你。走得太远意味着你超出了原则的“范围”，并且没有得到预期的优势。
> 原则、启发式、“工程法则”就像提示符号，当你知道它们的含义且知道何时你走得太远时，它们很有用。应用它们需要经验，即尝试、失败、分析、与人交谈、再次失败、修复、学习和接着失败。据我所知，没有捷径可走。

**在实现任何模式之前，始终分析使用它带来的好处是否值得我们承担额外的代码复杂性。**

> Effective design argues that we need to know the price of a pattern is worth paying - that's its own skill.

> 有效的设计提倡我们需要了解是否值得支付模式带来的开销 - 

不要仅仅因为书本和文章这么说就盲从实践、模式和架构。有时从头开始重写软件是最好的解决方案，而你将所有你知道的模式和架构风格融入项目中的所有努力都是浪费时间。尝试评估你实施的每个模式的成本和收益并避免过度工程。请记住架构、模式和原则是你在某些情况下有助于你的工具，不是你必须盲目遵循的教条。

然而，请记住：

> 重构过度设计比重构没有设计更容易。

阅读更多：

- [Martin Fowler blog: Yagni](https://martinfowler.com/bliki/Yagni.html)
- [7 Software Development Principles That Should Be Embraced Daily](https://betterprogramming.pub/7-software-development-principles-that-should-be-embraced-daily-c26a94ec4ecc?gi=3b5b298ddc23)
- [SOLID Principles and the Arts of Finding the Beach](https://sebastiankuebeck.wordpress.com/2017/09/17/solid-principles-and-the-arts-of-finding-the-beach/)

## 行为测试(Behavioral Testing)

行为测试（或者叫 [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development)）是对程序外部行为的测试，也称为黑盒测试。

具有通用语言的领域驱动设计与行为测试可以很好的结合在一起。

对于 BDD 测试，[Cucumber](https://cucumber.io/) 配合 [Gherkin](https://cucumber.io/docs/gherkin/reference/) 语法可以为你的测试提供结构和语义。这样，即使非开发人员也可以定义测试所需的步骤。在 node.js 中 [jest-cucumber](https://www.npmjs.com/package/jest-cucumber) 是一个不错的选项。

实例文件：

- [create-user.feature](https://github.com/Sairyss/domain-driven-hexagon/blob/master/tests/user/create-user/create-user.feature) - 包含人类可读 Gherkin 步骤的功能文件。
- [create-user.e2e-spec.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/tests/user/create-user/create-user.e2e-spec.ts) - e2e / 行为测试

阅读更多：

- [Backend best practices - Testing](https://github.com/Sairyss/backend-best-practices#testing)

## 目录和文件结构(Folder and File Structure)

因此，当整个应用程序被划分成服务、控制器等时，我们不是用典型的分层方式，而是将所有内容按模块划分。现在，如何在这些模块中处理文件结构？

很多人倾向于做和以前一样的事情：为一个模块创建一个大型的服务/控制器，并在这里维护模块用例的所有逻辑，使这些控制器和服务的代码长达数百行，这会导致文件难以查找并使合并冲突成为代码管理的噩梦。或者是为每种文件类型创建一个目录，例如 `interfaces` 或 `services` 目录，并在其中存储所有彼此无关的接口/服务。这种做法同样会使文件难以查找。每次当你需要修改点什么的时候，你需要在多个目录间跳转来查找相关文件而不是在同一个地方查找它们。

将每个模块按组件分开且将所有相关文件放在一起会更合乎逻辑。例如，查看 [create-user](src/modules/user/commands/create-user) 目录。这个目录包含了大部分与它相关的文件：控制器、服务、命令等。现在如果一个用例更新了，大部分的变更都将在这个单独的组件（目录）中进行，而不会扩散到整个模块的其他地方。

共享文件，例如领域对象（实体/聚合）、存储库、共享 DTOs 和接口等被分开存放，因为它们被多个用例重用。领域层是隔离的，且那些本质上是业务逻辑包装器的用例会被认为是组件。这种方法使文件查找和维护更容易。检查 [user](src/modules/user)模块查看更多例子。

这叫做 [The Common Closure Principle (CCP)](https://ericbackhage.net/clean-code/the-common-closure-principle/)。本项目中的目录/文件结构使用了这个原则。通常一起发生改变的相关文件（并且不被该组件之外的任何其他东西使用）被放到一起，在一个单独的用例目录中。

> 这里的目标应该是战略性的，并且根据经验，我们知道用一个组件中的这些类经常一起变化。

请记住，这个项目中的目录/文件结构只是一个示例，可能并不适合所有人。这里的主要建议是：

- 将你的应用程序划分为模块；
- 将会同时发生变更的文件放到一起(_Common Closure Principle_);
- 按行为是否一起改变对文件进行分组，而不是按文件的功能类型；
- 将那些会被多个组件复用的文件单独存放；
- 尊重你代码中的边界，将文件放在一起并不意味着内层可以导入外层；
- [Move files around until it feels right](https://dev.to/dance2die/move-files-around-until-it-feels-right-2lek).

这里有一些不同的方法来处理文件/目录结构，比如明确地将每一层分离到一个相应的目录中。这会更清晰地定义边界但会使文件查找变得困难。请选择更适合项目/个人偏好的方式。

例子：

- [Commands](src/modules/user/commands) 目录包含所有会导致状态变更的用例且每个用例都包含大部分它们需要的东西：控制器、服务、DTO、命令等。
- [Queries](src/modules/user/queries) 目录和命令目录差不多，但包含了数据检索用例。

阅读更多：

- [Out with the Onion, in with Vertical Slices](https://medium.com/@jacobcunningham/out-with-the-onion-in-with-vertical-slices-c3edfdafe118)
- [Vertical Slice Architecture](https://jimmybogard.com/vertical-slice-architecture/)

### 文件名(File names)

考虑在命名文件时在点号 "`.`"后使用一个描述性的类型名，例如 `*.service.ts` 或 `*.entity.ts`。这使得区分哪些文件做什么更容易，并且使用 [模糊搜索(fuzzy search)](https://en.wikipedia.org/wiki/Approximate_string_matching) (可以尝试在Windows/Linux 使用 `CTRL+P` ，MacOS 使用 `⌘+P`) 来搜索文件更容易。

阅读更多：

- [Angular Style Guides: Separate file names with dots and dashes](https://angular.io/guide/styleguide#separate-file-names-with-dots-and-dashes).

## 自定义实用类型(Custom utility types)

考虑为不同情况创建一些共享的自定义实用类型。

可以在 [types](src/libs/types) 目录下找到一些例子。

## 避免大规模继承链(Prevent massive inheritance chains)

这可以通过将类设为 `final` 来实现。

**注意**：在 TypeScript 中，和其他语言不同，没有默认方式来声明类是 `final` 的。但可以使用一个自定义装饰器来实现。

示例文件： [final.decorator.ts](src/libs/decorators/final.decorator.ts)

阅读更多：

- [When to declare classes final](https://ocramius.github.io/blog/when-to-declare-classes-final/)
- [Final classes by default, why?](https://matthiasnoback.nl/2018/09/final-classes-by-default-why/)
- [Prefer Composition Over Inheritance](https://medium.com/better-programming/prefer-composition-over-inheritance-1602d5149ea1)

---

# 其他资源(Additional resources)

查看这个仓库以获取更多这里使用的最佳实践：[Backend best practices](https://github.com/Sairyss/backend-best-practices)。

## 文章(Articles)

- [DDD, Hexagonal, Onion, Clean, CQRS, … How I put it all together](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together)
- [Hexagonal Architecture](https://www.qwan.eu/2020/08/20/hexagonal-architecture.html)
- [Clean architecture series](https://medium.com/@pereiren/clean-architecture-series-part-1-f34ef6b04b62)
- [Clean architecture for the rest of us](https://pusher.com/tutorials/clean-architecture-introduction)
- [An illustrated guide to 12 Factor Apps](https://www.redhat.com/architect/12-factor-app)

## Github仓库(Github Repositories)

- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [The System Design Primer](https://github.com/donnemartin/system-design-primer)

## 文档站点(Documentation Websites)

- [The Twelve-Factor App](https://12factor.net/)
- [Refactoring guru - Catalog of Design Patterns](https://refactoring.guru/design-patterns/catalog)
- [Microsoft - Cloud Design Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/index-patterns)

## 博客(Blogs)

- [Vladimir Khorikov](https://enterprisecraftsmanship.com/)
- [Khalil Stemmler](https://khalilstemmler.com)
- [Kamil Grzybek](https://www.kamilgrzybek.com/)
- [Martin Fowler](https://martinfowler.com/)
- [Herberto Graca](https://herbertograca.com/)

## 视频(Videos)

- [More Testable Code with the Hexagonal Architecture](https://youtu.be/ujb_O6myknY)
- [Playlist: Design Patterns Video Tutorial](https://youtube.com/playlist?list=PLF206E906175C7E07)
- [Playlist: Design Patterns in Object Oriented Programming](https://youtube.com/playlist?list=PLrhzvIcii6GNjpARdnO4ueTUAVR9eMBpc)
- [Herberto Graca - Making architecture explicit](https://www.youtube.com/watch?v=_yoZN9Sb3PM&feature=youtu.be)

## 书籍(Books)

- ["Domain-Driven Design: Tackling Complexity in the Heart of Software"](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by Eric Evans
- ["Secure by Design"](https://www.manning.com/books/secure-by-design) by Dan Bergh Johnsson, Daniel Deogun, Daniel Sawano
- ["Implementing Domain-Driven Design"](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) by Vaughn Vernon
- ["Clean Architecture: A Craftsman's Guide to Software Structure and Design"](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?dchild=1&keywords=clean+architecture&qid=1605343702&s=books&sr=1-1) by Robert Martin
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321) by Martin Kleppmann
