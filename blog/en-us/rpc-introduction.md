---
Title: Talking about RPC
Keywords: RPC, Dubbo, Spring Cloud
Description: RPC - Remote Procedure Call, a protocol that requests services from a remote computer program over a network without the need to understand the underlying network technology.
---

In recent years, with the rise of micro-service projects, it has gradually become the mainstream of large and medium-sized distributed system architectures in many companies, and today RPC plays a vital role in this. With the evolution of the micro-services of the company's projects during this period, it is found that RPC is implicitly or explicitly used in daily development. Some small partners who have just contacted RPC will feel at a loss, and some veterans who have been in the industry for many years use RPC experience. Rich, but some have a little understanding of its principles, lack of in-depth understanding of the principle, often lead to some misuse in development.

## What is RPC?

RPC (Remote Procedure Call) - A remote procedure call, which is a protocol that requests services from a remote computer program over a network without the need to understand the underlying network technology. That is to say, two servers A, B, an application deployed on the A server, want to call the method provided by the application on the B server, because it is not in a memory space, can not be directly called, need to express the semantics of the call and communicate the call through the network The data.

The RPC protocol assumes that certain transport protocols exist, such as TCP or UDP, to carry informational data between communication programs. In the OSI network communication model, RPC spans the transport layer and the application layer. RPC makes it easier to develop applications that include network-distributed multi-programs. There are many open source RPC frameworks in the industry, such as Spring Cloud, Dubbo, Thrift, etc.

## RPC Origin

The term RPC was coined by **Bruce Jay Nelson** in the 1980s. Here we trace the original motives for the development of RPC. In Nelson's paper "Implementing Remote Procedure Calls" he mentioned a few points:
* Simplicity: The semantics of the RPC concept are very clear and simple, making it easier to build distributed computing.
* Efficient: Process calls seem simple and efficient.
* General: In stand-alone computing, the process is often the most important communication mechanism between different algorithm parts.

To put it bluntly, the average programmer is familiar with local procedure calls. So we make the RPC completely similar to the local call, so it is easier to accept and use without any obstacles. Nelson's paper was published 30 years ago, and its views seem to be far-sighted today. The RPC framework we use today is basically achieved according to this goal.

## RPC structure

Nelson's paper states that the program for implementing RPC consists of five parts:
1. User
2. User-stub
3. RPCRuntime
4. Server-stub
5. Server

![RPC Structure](../../img/blog/rpc/rpc-structure-1.png)

Here user is the client side. When user wants to initiate a remote call, it actually calls user-stub locally. The user-stub is responsible for encoding the called interfaces, methods, and parameters through the agreed protocol specification and transmitting them to the remote instance through the local RPCRuntime instance. After receiving the request, the remote RPCRuntime instance sends the request to the server-stub for decoding, and then initiates the local call. The result is returned to the user.

The above is a coarse-grained RPC implementation conceptual structure. Next we will further refine what components it should consist of, as shown in the following figure.

![RPC Structure Disassembly](../../img/blog/rpc/rpc-structure-2.png)

The RPC server exports the remote interface method through RpcServer, and the client uses the RpcClient to import the remote interface method. The client invokes the remote interface method just like the native method. The RPC framework provides the proxy implementation of the interface, and the actual call is delegated to the proxy RpcProxy. The proxy encapsulates the call information and forwards the call to RpcInvoker for actual execution. The client's RpcInvoker maintains the channel RpcChannel with the server through the connector RpcConnector, and uses RpcProtocol to perform protocol encoding (encode) and sends the encoded request message to the servant through the channel.

RPC server receiver RpcAcceptor receives the client's call request, and also uses RpcProtocol to perform protocol decoding (decode). The decoded call information is passed to RpcProcessor to control the processing of the call process, and finally the call is called to RpcInvoker to actually execute and return the call result. The following are the detailed responsibilities of each part:

```
1. RpcServer

   Responsible for exporting the remote interface

2. RpcClient

   The agent implementation responsible for importing the remote interface

3. RpcProxy

   Proxy implementation of the remote interface

4. RpcInvoker

   Client side implementation: responsible for encoding the call information and sending the call request to the servant and waiting for the call result to return

   Provider implementation: responsible for calling the concrete implementation of the server interface and returning the result of the call

5. RpcProtocol

   Responsible for protocol editing/decoding

6. RpcConnector

   Responsible for maintaining the connection channel between the client and the service party and sending data to the service party

7. RpcAcceptor

   Responsible for receiving client requests and returning request results

8. RpcProcessor

   Responsible for controlling the calling process on the servant, including managing the calling thread pool, timeout, etc.

9. RpcChannel

   Data transmission channel
```

## RPC working principle

The design of the RPC consists of Client, Client stub, Network, Server stub, and Server. The Client is used to call the service. The Cient stub is used to serialize the methods and parameters of the call (because the object has to be converted to bytes in the network), Network is used to transfer this information to the Server stub. The Server stub is used to deserialize this information. The Server is the provider of the service, and the final method is the method provided by the Server.

![How RPC works] (../../img/blog/rpc/rpc-work-principle.png)

1. The client calls the remote service like a local service;

2. After the client stub receives the call, serialize the method and parameters.

3. The client sends the message to the server through the sockets.

4. The server stub decodes after receiving the message (deserializes the message object)

5. The server stub calls the local service based on the decoding result.

6. Local service execution (local execution for the server) and return the result to the server stub

7. The server stub packages the returned result into a message (serializes the resulting message object)

8. The server sends the message to the client through the sockets.

9. The client stub receives the result message and decodes it (serializes the resulting message)

10. The client gets the final result.

There are two types of RPC calls:

1. Synchronous call: The client waits for the call to complete and returns the result.

2. Asynchronous call: After the client invokes, it does not have to wait for the execution result to return, but the return result can still be obtained through callback notification. If the client does not care about the result of the call, it becomes a one-way asynchronous call, and the one-way call does not return the result.

The difference between asynchronous and synchronous is whether to wait for the server to execute and return the result.
   
## RPC What can I do?

The main functional goal of RPC is to make it easier to build distributed computing (applications) without compromising the semantic simplicity of local calls when providing powerful remote invocation capabilities. To achieve this goal, the RPC framework needs to provide a transparent invocation mechanism that allows the user to explicitly distinguish between local and remote calls. An implementation structure given earlier, based on the stub structure. Below we will specifically refine the implementation of the stub structure.

* Can be distributed, modern microservices
* Flexible deployment
* Decoupling service
* Strong scalability

The purpose of RPC is to let you call remote methods locally, and for you this call is transparent, you don't know where the method of the call is deployed. The decoupling of services through RPC is the real purpose of using RPC.

## to sum up

This article introduces some of the basic principles of RPC. I believe that you have some understanding of RPC here. In fact, it is not difficult to realize an RPC. It is difficult to implement a high-performance and reliable RPC framework. For example, since it is distributed, a service may have multiple instances. How do you get the addresses of these instances when you call them? At this time, you need a service registry. For example, in Dubbo, you can use Zookeeper as the registration center. When calling, you can get a list of instances of the service from Zookeeper and select one to call. So which one to call is good? At this time, load balancing is required, so you have to consider how to implement complex balancing. For example, Dubbo provides several load balancing strategies. So please continue to pay attention to my other two articles ** RPC and service relationship ** and ** registration center, configuration center, service discovery **, I believe will help more understanding of RPC design and implementation.
