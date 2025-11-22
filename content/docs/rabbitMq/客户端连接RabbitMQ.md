客户端连接RabbitMQ
<!-- more -->
# RabbitMQ.Client

基于 `RabbitMQ.Client` 的封装，在 `NuGet` 中搜索 `RabbitMQ.Client` ，直接点击按钮安装即可。

```csharp
namespace CommonLib.RabbitMQ
{
    using global::RabbitMQ.Client;
    using global::RabbitMQ.Client.Events;

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading;

    public class QueueOptions
    {
        /// <summary>
        /// 是否持久化
        /// </summary>
        public bool Durable { get; set; } = true;

        /// <summary>
        /// 是否自动删除
        /// </summary>
        public bool AutoDelete { get; set; } = false;

        /// <summary>
        /// 参数
        /// </summary>
        public IDictionary<string, object> Arguments { get; set; } = new Dictionary<string, object>();
    }

    public class ConsumeQueueOptions : QueueOptions
    {
        /// <summary>
        /// 是否自动提交
        /// </summary>
        public bool AutoAck { get; set; } = false;

        /// <summary>
        /// 每次接收消息条数
        /// </summary>
        public ushort? FetchCount { get; set; } = 1;
    }

    public class ExchangeConsumeQueueOptions : ConsumeQueueOptions
    {
        /// <summary>
        /// 路由值
        /// </summary>
        public string[] RoutingKeys { get; set; }

        /// <summary>
        /// 参数
        /// </summary>
        public IDictionary<string, object> BindArguments { get; set; } = new Dictionary<string, object>();
    }

    public class ExchangeQueueOptions : QueueOptions
    {
        /// <summary>
        /// 交换机类型
        /// </summary>
        public string Type { get; set; }

        /// <summary>
        /// 队列及路由值
        /// </summary>
        public List<Tuple<string, string>> QueueAndRoutingKey { get; set; } = new List<Tuple<string, string>>();

        /// <summary>
        /// 参数
        /// </summary>
        public IDictionary<string, object> BindArguments { get; set; } = new Dictionary<string, object>();
    }

    public static class RabbitMQExchangeType
    {
        /// <summary>
        /// 普通模式
        /// </summary>
        public const string Common = "";

        /// <summary>
        /// 路由模式
        /// </summary>
        public const string Direct = "direct";

        /// <summary>
        /// 发布/订阅模式
        /// </summary>
        public const string Fanout = "fanout";

        /// <summary>
        /// 匹配订阅模式
        /// </summary>
        public const string Topic = "topic";
    }

    public abstract class RabbitBase : IDisposable
    {
        private readonly List<AmqpTcpEndpoint> amqpList;
        private IConnection connection;

        protected RabbitBase(params string[] hosts)
        {
            if (hosts == null || hosts.Length == 0)
            {
                throw new ArgumentException("invalid hosts！", nameof(hosts));
            }

            amqpList = new List<AmqpTcpEndpoint>();
            amqpList.AddRange(hosts.Select(host => new AmqpTcpEndpoint(host, Port)));
        }

        protected RabbitBase(params (string, int)[] hostAndPorts)
        {
            if (hostAndPorts == null || hostAndPorts.Length == 0)
            {
                throw new ArgumentException("invalid hosts！", nameof(hostAndPorts));
            }

            amqpList = new List<AmqpTcpEndpoint>();
            amqpList.AddRange(hostAndPorts.Select(tuple => new AmqpTcpEndpoint(tuple.Item1, tuple.Item2)));
        }

        /// <summary>
        /// 端口
        /// </summary>
        public int Port { get; set; } = 5672;

        /// <summary>
        /// 账号
        /// </summary>
        public string UserName { get; set; } = ConnectionFactory.DefaultUser;

        /// <summary>
        /// 密码
        /// </summary>
        public string Password { get; set; } = ConnectionFactory.DefaultPass;

        /// <summary>
        /// 虚拟机
        /// </summary>
        public string VirtualHost { get; set; } = ConnectionFactory.DefaultVHost;

        public virtual void Dispose()
        {
            // connection?.Close();
            // connection?.Dispose();
        }

        /// <summary>
        /// 关闭连接
        /// </summary>
        public void Close()
        {
            connection?.Close();
            connection?.Dispose();
        }

        /// <summary>
        /// 获取rabbitmq的连接
        /// </summary>
        /// <returns></returns>
        protected IModel GetChannel()
        {
            if (connection == null)
            {
                lock (this)
                {
                    if (connection == null)
                    {
                        ConnectionFactory factory = new()
                        {
                            Port = Port,
                            UserName = UserName,
                            VirtualHost = VirtualHost,
                            Password = Password
                        };
                        // 网络故障自动恢复连接
                        factory.AutomaticRecoveryEnabled = true;
                        // 心跳处理
                        factory.RequestedHeartbeat = new TimeSpan(5000);
                        connection = factory.CreateConnection(amqpList);
                    }
                }
            }
            return connection.CreateModel();
        }
    }

    public class RabbitMQHelper : RabbitBase
    {
        public RabbitMQHelper(params string[] hosts) : base(hosts)
        {

        }

        public RabbitMQHelper(params (string, int)[] hostAndPorts) : base(hostAndPorts)
        {

        }

        /// <summary>
        /// 简单队列消息发布
        /// </summary>
        /// <param name="queue">The queue.</param>
        /// <param name="message">The message.</param>
        /// <param name="options">The options.</param>
        public void Publish(string queue, string message, QueueOptions options = null)
        {
            options ??= new QueueOptions();
            IModel channel = GetChannel();
            channel.QueueDeclare(queue, options.Durable, false, options.AutoDelete, options.Arguments ?? new Dictionary<string, object>());
            byte[] buffer = Encoding.UTF8.GetBytes(message);
            channel.BasicPublish(exchange: "", routingKey: queue, basicProperties: null, body: buffer);
            channel.Close();
        }

        /// <summary>
        /// 订阅模式/路由模式/Topic模式
        /// </summary>
        /// <param name="exchange">The exchange.</param>
        /// <param name="routingKey">The routing key.</param>
        /// <param name="message">The message.</param>
        /// <param name="options">The options.</param>
        public void Publish(string exchange, string routingKey, string message, ExchangeQueueOptions options = null)
        {
            options ??= new ExchangeQueueOptions();
            IModel channel = GetChannel();

            channel.ExchangeDeclare(exchange,
                string.IsNullOrEmpty(options.Type) ? RabbitMQExchangeType.Fanout : options.Type,
                options.Durable,
                options.AutoDelete,
                options.Arguments ?? new Dictionary<string, object>());

            if (options.QueueAndRoutingKey.Count > 0)
            {
                foreach (var item in options.QueueAndRoutingKey)
                {
                    if (!string.IsNullOrEmpty(item.Item1))
                    {
                        channel.QueueDeclare(item.Item1, options.Durable, false, options.AutoDelete, options.Arguments);

                        channel.QueueBind(item.Item1,
                            exchange,
                            item.Item2 ?? "",
                            options.BindArguments ?? new Dictionary<string, object>());
                    }
                }
            }
            byte[] buffer = Encoding.UTF8.GetBytes(message);
            channel.BasicPublish(exchange, routingKey, null, buffer);
            channel.Close();
        }

        public event Action<RecieveResult> Received;

        /// <summary>
        /// 构造消费者
        /// </summary>
        /// <param name="channel"></param>
        /// <param name="options"></param>
        /// <returns></returns>
        private IBasicConsumer ConsumeInternal(IModel channel, ConsumeQueueOptions options)
        {
            EventingBasicConsumer consumer = new(channel);
            consumer.Received += (sender, e) =>
            {
                try
                {
                    CancellationTokenSource cancellationTokenSource = new();
                    if (!options.AutoAck)
                    {
                        cancellationTokenSource.Token.Register(() =>
                        {
                            channel.BasicAck(e.DeliveryTag, false);
                        });
                    }
                    Received?.Invoke(new RecieveResult(e, cancellationTokenSource));
                }
                catch { }
            };
            if (options.FetchCount != null)
            {
                channel.BasicQos(0, options.FetchCount.Value, false);
            }
            return consumer;
        }

        /// <summary>
        /// 消息监听
        /// </summary>
        /// <param name="queue">The queue.</param>
        /// <param name="options">The options.</param>
        /// <returns></returns>
        public ListenResult Listen(string queue, ConsumeQueueOptions options = null)
        {
            options ??= new ConsumeQueueOptions();
            IModel channel = GetChannel();
            channel.QueueDeclare(queue, options.Durable, false, options.AutoDelete, options.Arguments ?? new Dictionary<string, object>());
            IBasicConsumer consumer = ConsumeInternal(channel, options);
            channel.BasicConsume(queue, options.AutoAck, consumer);
            ListenResult result = new();
            result.Token.Register(() =>
            {
                try
                {
                    channel.Close();
                    channel.Dispose();
                }
                catch { }
            });
            return result;
        }

        /// <summary>
        /// 消费消息
        /// </summary>
        /// <param name="exchange"></param>
        /// <param name="queue"></param>
        /// <param name="options"></param>
        public ListenResult Listen(string exchange, string queue, ExchangeConsumeQueueOptions options = null)
        {
            options ??= new ExchangeConsumeQueueOptions();
            IModel channel = GetChannel();
            channel.QueueDeclare(queue, options.Durable, false, options.AutoDelete, options.Arguments ?? new Dictionary<string, object>());
            if (options.RoutingKeys != null && !string.IsNullOrEmpty(exchange))
            {
                foreach (string key in options.RoutingKeys)
                {
                    channel.QueueBind(queue, exchange, key, options.BindArguments);
                }
            }
            IBasicConsumer consumer = ConsumeInternal(channel, options);
            channel.BasicConsume(queue, options.AutoAck, consumer);
            ListenResult result = new();
            result.Token.Register(() =>
            {
                try
                {
                    channel.Close();
                    channel.Dispose();
                }
                catch { }
            });
            return result;
        }

        public class RecieveResult
        {
            private CancellationTokenSource cancellationTokenSource;

            public RecieveResult(BasicDeliverEventArgs arg, CancellationTokenSource cancellationTokenSource)
            {
                Body = Encoding.UTF8.GetString(arg.Body.ToArray());
                ConsumerTag = arg.ConsumerTag;
                DeliveryTag = arg.DeliveryTag;
                Exchange = arg.Exchange;
                Redelivered = arg.Redelivered;
                RoutingKey = arg.RoutingKey;
                this.cancellationTokenSource = cancellationTokenSource;
            }

            /// <summary>
            /// 消息体
            /// </summary>
            public string Body { get; private set; }

            /// <summary>
            /// 消费者标签
            /// </summary>
            public string ConsumerTag { get; private set; }

            /// <summary>
            /// Ack标签
            /// </summary>
            public ulong DeliveryTag { get; private set; }

            /// <summary>
            /// 交换机
            /// </summary>
            public string Exchange { get; private set; }

            /// <summary>
            /// 是否Ack
            /// </summary>
            public bool Redelivered { get; private set; }

            /// <summary>
            /// 路由
            /// </summary>
            public string RoutingKey { get; private set; }

            public void Commit()
            {
                if (cancellationTokenSource == null || cancellationTokenSource.IsCancellationRequested) return;

                cancellationTokenSource.Cancel();
                cancellationTokenSource.Dispose();
                cancellationTokenSource = null;
            }
        }
        public class ListenResult
        {
            private readonly CancellationTokenSource cancellationTokenSource;

            /// <summary>
            /// CancellationToken
            /// </summary>
            public CancellationToken Token { get { return cancellationTokenSource.Token; } }

            /// <summary>
            /// 是否已停止
            /// </summary>
            public bool Stoped { get { return cancellationTokenSource.IsCancellationRequested; } }

            public ListenResult()
            {
                cancellationTokenSource = new CancellationTokenSource();
            }

            /// <summary>
            /// 停止监听
            /// </summary>
            public void Stop()
            {
                cancellationTokenSource.Cancel();
            }
        }
    }
}
```

# EasyNetQ

## 简介

EasyNetQ 是基于官方.NET组件 RabbitMQ.Client 的又一层封装，使用起来更加方便，不用关心具体队列声明，路由声明等细节。EasyNetQ目的是提供一个尽可能简洁适用于RabbitMQ的.NET类库。为了实现这些目标，EasyNetQ强制使用了一些简单的约定。包括如下:

- 消息用 .NET 类型 表示
- 消息通过 .NET类型 路由

这意味着消息必须用 .NET class定义。每一个不同的消息类型必须用一个 class 表示。这个类必须是 public 并带有一个默认构造函数和可以读写的属性。这个类不需要实现任何功能。仅仅做一个简单的数据容器，下面是一个简单的消息：

```csharp
public class MyMessage
{
   public string Content { get; set; }
}
```

EasyNetQ通过消息的类型来路由。当发布一个消息，EasyNetQ会检查消息类型， 然后给它一个 基于类型名称、命名空间和程序集的路由键。默认EasyNetQ使用 Newtonsoft.Json 序列化 .NET类型 为 JSON ，这样好处是消息可读性好。

> **引用**：EasyNetQ是一个在RabbitMQ.Client类库之上提供服务的组件集合。做了这些事情，像序列化、错误处理、线程管理、连接管理等。通过一个Mini-Ioc容器组织在一起。你能很容易用你自己实现去替换这些组件。所以如果你喜欢用XML 序列化而不是用JSON，仅仅需要以一个ISerializer的实现，然后注册到这个容器中。
>
> 这些组件最上层是IAdvancedBus API。这看起来很像AMQP规格。实际也是你能够通过这个API运行很多AMQP方法。这个API对你隐藏了唯一AMQP概念是channels。这是因为channels 是一个复杂的底层概念，不应该被放到AMQP部分规格的第一的位置。 坦白来说，这个API中 ‘Advanced’不是一个非常好的名字。用‘lamqp’可能更好些。
>
> 这个顶层高级API是一系列消息模式：Publish/Subscribe, Request/Response,和 Send/Receive. 这是EasyNetQ坚持的设计思想。这些模式是我们应该实现的。这样有非常小的弹性。要么你接受我的处理方法，或者你就不要去使用。这样做的目的是，不用你和使用者花费精力去重新发明轮子。你不需要每一次去做选择，你只需要简单的去Publish和Subscribe消息。这样设计是未来实现EasyNetQ的核心目标，即尽可能简单的使用RabbitMQ。
>
> 这些模式的后面是这个 IBus API. 再一次看到这个一个简单的名字，它跟消息总线概念有关。IPackagedMessagePatterns可能是一个更好名字。
>
> 80%的用户的工作，在80%的时间都会使用IBus。它不是完备的API，如果这个模式下，你想实现的功能这个IBus没有提供，那么你应该使用IAdvancedBus。这样使用没有问题，EasyNetQ就这这样设计使用的。

## 优势

C#中已经提供了RabbitMQ.Client， RabbitMQ. Client 实现了AMQP协议的客户端（RabbitMQ实现了服务器端）。 AMQP是为HTTP协议设计的。它的设计是跨平台的和与语言无关的。它也旨在灵活支持多种基于交换/绑定/队列模型的消息传递模式。

RabiitMQ Client 非常地灵活，但是伴随着灵活性而来是复杂性。这意味着需要写大量代码。比如：

- 实现消息传递模式，例如 `Publish/Subscribe`或 `Request/Response` 
- 实现路由策略。需要设计如何绑定 Exchange/Queue 。并且需要设计怎样在生产者和消费者之间进行消息路由
- 实现消息的序列化/反序列化。 如何转换AMQP的二进制消息为编程语言能理解的格式
- 为订阅去实现一个消费者线程。将需要有一个专门的消费者循环等待订阅的消息。如何处理多个订阅者，或者瞬间订阅者
- 实现消费者重新连接。假如连接崩溃了或者RabbitMQ 服务挂了，怎样能检测到并确保所有的订阅都能被重建
- 懂得和实施服务质量设置。需要什么样的设置来确保一个可靠的客户端
- 实现一个错误处理策略。假如接受到一个错误的消息，或者发生一个未处理异常被抛出，客户端应该做什么
- 实现发布者可靠的消息确认。

EasyNetQ目标是在AMQP之上封装所有这些关注点在一个简单好用的类库中。

## 安装

在 `NuGet` 中搜索 `EasyNetQ` 直接点击按钮安装即可。EasyNetQ 依赖 RabbitMQ.Client，所以会同时安装两个dll。

## 连接

使用 EasyNetQ 连接 RabbitMQ，是在应用程序启动时创建一个 `IBus` 对象，并且在应用程序关闭时释放该对象。`RabbitMQ` 连接是基于 `IBus` 接口的，当`IBus` 中的方法被调用，连接才会开启。创建一个 `IBus` 对象的方法如下：

```csharp
var bus = RabbitHutch.CreateBus(“host=myServer;virtualHost=myVirtualHost;username=mike;password=topsecret”);
```

连接字符串基于 `Key/Value` 形式，每个Key中间用分号 `;` 断开。其中 `host` 是必须的，其他值采用默认配置。连接中可能用到的 `Key` 如下：

- `host`：`host=localhost` 或 `host =192.168.1.102` 或者 `host=my.rabbitmq.com` ,集群配置的话可以用逗号将服务地址隔开，例如：`host=a.com,b.com,c.com`
- `virtualHost`：虚拟主机，默认 `/`
- `username`：登录名
- `password`：登录密码
- `requestedHeartbeat`：心跳设置，默认10秒
- `prefetchcount`：默认是50
- `pubisherConfirms`：默认 `false`
- `persistentMessages`：消息持久化，默认 `true`
- `product`：产品名
- `platform`：平台
- `timeout`：默认10秒

关闭连接，使用 `bus.Dispose();`

## 日志

EasyNetQ 提供了日志接口 ` IEasyNetQLogger`

```csharp
public interface IEasyNetQLogger
{
   void DebugWrite(string format, params object[] args);
   void InfoWrite(string format, params object[] args);
   void ErrorWrite(string format, params object[] args);
   void ErrorWrite(Exception exception);
}
```

内部默认用的是 `NullLogger`，即什么也不做，不记录日志。在测试的时候也可以用 `ConsoleLogger` 来显示运行中的各种信息。不过一般在正式使用环境中，可以自定义日志并实现 `IEasyNetQLogger`接口。然后在 `RabbitHutch.CreateBus` 的重载方法中注册想用的日志类型。（日志中会记录连接RabbitMQ的过程和队列创建细节等信息）代码如下：

```csharp
var logger = new MyLogger() // 继承自 IEasyNetQLogger
var bus = RabbitHutch.CreateBus(“my connection”, x => x.Register<IEasyNetQLogger>(_ => logger));
```

## 发布/订阅

EasyNetQ 支持最简单的消息模式是：发布/订阅。发布消息后任意消费者都可以订阅该消息，并且不需要额外配置。首先：需要先创建一个 `IBus` 对象，然后创建一个**可序列化的 `.NET`对象**。调用 `Publish`方法即可。

**Pub**

```csharp
services.AddSingleton(RabbitHutch.CreateBus(redisConnection));
IServiceProvider provider = services.BuildServiceProvider();
mq = provider.GetService<IBus>();
mq.PubSub.Publish(new { message = "hello.world" });
```

> 注意：Publish只顾发送消息到队列，但是不管有没有消费端订阅，所以，发布之后，如果没有消费者，该消息将不会被消费甚至丢失。

**Sub**

一个EasyNetQ订阅者订阅一种消息类型（消息类为.NET 类型）。通过调用Subcribe方法一旦对一个类型设置了订阅，一个持久化队列就会在RabbitMQ broker上被创建，这个类型的任何消息都会被发送到这个队列上。

```
 mq.PubSub.Subscribe<MyMessage>("", msg =>
 {
       Console.WriteLine(msg.Content);
 });
```

**subscription_id**

**相同类型消息、相同订阅id调用Subscribe**：EasyNetQ将会在RabbitMQ Broker上为特定的消息类型的和订阅id的组合创建唯一的队列。每一次调用Subscribe方法会创建一个新的队列消费者。如果用相同的消息和订阅id调用Subscribe两次，将会创建两个消费者去消费同一个队列。然后RabbitMQ将会依次连续轮询消息给每一个消费者（均摊机制）。这种可伸缩性和工作分担是非常棒的。比如：一个处理消息的服务已经超负荷工作了。简单的创建一个新的服务实例（在同一个机器上，或者不同的机器上），不用配置任何东西，自动就得到了伸缩性。

**相同类型消息、不同订阅id调用Subscribe**：假如相同的消息类型，用不同的订阅id调用了两次Subscribe，将创建两个队列，每一个队列有自己的消费者。每一个消息的副本将会路由到每一个队列，因此不同的消费者都将得到所有消息（这个类型的，Fanout）。适用于几个不同的服务都关心相同类型的消息。



