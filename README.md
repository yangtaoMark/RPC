# 一个简单的PRC框架实现

    组员：杨涛，吴之玥，吴上，陶崇民
    杨涛(19020060)，陶崇民(18020088)：coding
    吴之玥(19020058)，吴上(19020041)：实验报告
    项目地址: https://github.com/yangtaoMark/RPC.git

## 工作流程

1. 本地调用某个函数方法

2. 本地机器的RPC框架把这个调用信息封装起来（调用的函数、入参等），序列化后，通过网络传输发送给远程服务器

3. 远程服务器收到调用请求后，远程机器的RPC框架反序列化获得调用信息，并根据调用信息定位到实际要执行的方法，执行完这个方法后，序列化执行结果，通过网络传输把执行结果发送回本地机器

4. 本地机器的RPC框架反序列化出执行结果，函数return这个结果
![RPC](./RPC/RPC.001.png)

## 基于操作系统的socket和多线程编程

对于远程调用来说，实际运行程序的被调用端，应同时应对多个服务请求，而且这些请求的执行不能相互干扰。

```python
    ip_port = ("127.0.0.1", 8000)
    s = socketserver.ThreadingTCPServer(ip_port, MyServer)
    s.serve_forever()
```

多线程的TCP服务器将会并行执行调用请求，提高可用率

## 基于JSON(JavaScript Object Notation)的IDL(interface description language)实现

JSON作为一种轻量级的数据交换语言，与XML一道，成为现代计算编程中最受欢迎的数据交换方式，它的跨语言特性十分适合作为RPC的IDL

```JSON
{
    "type": "request",
    "async":True,
    "functionName": functionName,
    "parameters": parameters
}
```

## 跨语言的并发调用

对于每一次调用，都会有一个唯一的$uuid$，作为返回结果的定位。我们提供了同步(sync)和异步(async)两种调用方式。同步(sync)调用将会在调用后维持长连接，等待返回结果。异步(async)调用将会立刻返回结果$uuid$，以便以后访问计算结果。由于没有设置主动断开机制，同步调用将是不建议的。

我们分别完成了python和node.js的远程调用框架，两两之间通过JOSN屏蔽了语言的异构性。其中，python的并发机制采用了多线程(multi threading),而node.js的并发机制采用了天然异步，两种不同的并发机制都可以满足要求，因此我们都实现了。

使用C++实现了RPC的客户端，在进行远程调用时只需提供方法名和参数即可，具有比较好的灵活性。在进行数据传输时，利用rapidjson将数据封装成json格式进行传输，实现了跨语言调用。接收服务端发来的返回结果，利用rapidjson解析出方法的返回值，最后返回给方法的调用者。

## 平台化

该远程调用仅仅依赖基础库，安装解释器即可。对于调用的修改，我们使用了语言的反射机制(refect)，仅仅需要在服务器端，增加该调用方法的实现即可。

```python

class MyServer(RPCServer):
    def add(self, parameters):
        print("add")
        time.sleep(10)
        result = 0
        for i in range(len(parameters)):
            result = result + parameters[i]
        print("add over")
        return result
```

例如，只需要通过继承基础PRC调用框架，增加调用方法即可


## 使用github 协作开发

    https://github.com/yangtaoMark/RPC

开发者

    yangtaoMark, dreaming0017

多分枝开发
    ![RPC](./RPC/branch.png)

分支合并
    ![RPC](./RPC/pullrequest.png)

## 课程实验建议

对于RPC简易框架实验，可否更加明确需求，而非简单设立标准。

如：

1. 不能使用现有的RPC框架，必须基于操作系统Socket接口编程，具有一定“平台化”特点
2. 不能只能调用一个特定方法
3. 比如至少能只需要改动一点点就用于另一个方法

以下不要求实现，但实现了可加分(可超出大作业环节的分数)

    • 跨语言调用能力 • IDL和IDL编译器 • 并发模型

修改为：

基于操作系统网络接口和基础库，完成并发的，可拓展的，跨语言，跨平台的无服务器云函数框架(Serverless Cloud Function)

无服务器(serverless)执行环境能够帮助用户在没有购买和管理服务器时仍能运行代码。用户只需要使用云平台支持的语言编写核心代码及设置代码运行的条件，代码即可在腾讯云基础设施上弹性、安全地运行，并可完全管理底层计算资源，包括服务器CPU、内存、网络、代码部署、弹性伸缩、负载均衡等服务。使用无服务器云函数将可免除所有运维性操作，企业和开发者可以更加专注于核心业务的开发，实现快速上线和迭代，把握业务发展的节奏。