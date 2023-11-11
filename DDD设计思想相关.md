### 方法的入参，实体的属性都应该是具有业务上的语义。
比如有一个String password。虽然看名字能够知道这是一个密码，但是也仅仅知道这是一个密码。
如果是Password Password。我们光是点到类里面一看，就可以看到跟密码相关的业务语义，一下就能把握到要点。


这样带来的好处有：
1. 可以很好的收敛参数校验的代码，不必掺杂在写业务的地方。


### DDD思想
高内聚，低耦合

总的来说，该做的工作是没有减少的，该有的复杂读也没有降低，但是将代码的位置编排的更加合理，使得结构更加卓越，就带来了可扩展性，易维护性等实实在在的好处。

### 事务脚本型代码
按照业务流程，将所有的业务所需要的数据啊，各种逻辑编排到一个service里面，是最简单最符合直觉的代码写法。

会带来几个问题：可维护性差、可扩展性差，可测试性差。

可维护性：当依赖变化的时候，有多少代码需要改动。

可扩展性：做新需求或者改逻辑时，需要新增/修改多少代码。

可测试性：运行每个单元测试用例所需要的时间*每个需求所需要增加的测试用例数量。

### DDD代码架构
传统的代码结构是从上到下的依赖，最顶层的controller，中间的service，最底下的dao。
DDD的代码结构是最外层的基础设施层（对各类基础设施进行直接IO的地方）、次外层防腐层（避免应用层直接依赖基础设施。靠近数据库一侧是data converter用来将do转换成entity，靠近接口一侧是data assember，用来将dto转换成entity）、内层的应用层、以及最核心的领域层。

越靠近核心，代码变化越灵活，越容易修改且被修改。

其实核心就是通过分层的思想，来隔绝直接依赖，从而进行依赖解耦，然后做到代码解耦层面的低耦合。

### 领域层
DDD最核心的层。通常由Entity、ValueObject、Domain Service。

Entity：包含ID和内部状态。是可以独立存在的。最重要的核心是保证实体的不变性。

Domain Service：用来辅助两个以上的Entity建立联系。

因为一个Entity不能直接参考另一个Entity或者服务，只能保存自己的状态或者（非聚合根对象）。一旦把其他对象弄进来了，就不好单测了。

Domain Service通常由三种类型：
1. 单对象策略性（用来实现单个对象的变更），通常通过参数传入Entity的方法，然后反调Service的方法来实现功能。
2. 跨对象事务型（会修改多个Entity的属性），这种就直接调用Service的方法，然后把受影响的Entity传进Service的方法。
3. 通用组件型。（可以用来修改一类对象的变更，比如实现同一个接口的若干个Entity），通常是把Entity传进Service的方法里来实现的。


### interface层
接口层负责与外界交互，主要应该具有以下功能：
1. 统一鉴权
2. session管理
3. 异常处理
4. 日志打印
5. 前置缓存

### Application层
Application层主要包含：application Service、DTO Assembler、CQE（Commend、Query、Event）、DTO。

CQE（Commend、Query、Event）都是Value Object，但是语义有不同，Commend指令是写操作，Query查询是只读操作，Event事件指一件一件发生过的事情，系统会伴随发生一些操作，大部分时候是写。

除了根据单一ID查询，Application Service的入参都应该是CQE，出参都是DTO

Application Service是业务流程的封装，不处理业务逻辑，只做编排。

DTO是将一个或者多个Entity封装而成的只有数据，没有任何逻辑的贫血模型类，用来与外界交互。

DTO Assembler是将一个或者多个Entity/VO转换成DTO

### Repository层
狭义上来讲，就是提供数据库操作的ACL防腐层，让核心领域层与操作数据库的基础设施层解耦。
广义上来讲，提供对所有的基础设施操作的防腐层，不仅仅是数据，还可能是外部接口，可能是各种消息服务中间件等等。


Repository层主要包含：Do、data mapper、data Converter。


Do是数据库表映射，贫血模型。
data mapper，就是传统三层架构中的mapper，比如mybatis那种。
data converter，主要是提供两个功能，一是讲DO转换成Entity/VO，二是将Entity/VO转换成DO。

### 复杂业务流程设计模式
Orchestration vs Choreography
Orchestration：命令驱动，上游强依赖下游，灵活性较差，上游为业务负责。
Choreography：事件驱动，无调用依赖，但是有代码依赖，可以算成是下游依赖上游。灵活性较强。没有全局负责人

具体选择哪种模式，根据业务要求来判断，如果上游要感知下游，那就是命令驱动，就是Orchestration。

反之上游不依赖下游，反而下游依赖上游的通知，则是事件驱动，应该选择Choreography





