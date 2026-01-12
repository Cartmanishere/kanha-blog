---
author: 'Pranav Gajjewar'
authorTwitter: ''
cover: '/images/understanding-grpc/grpc-intro-cover.jpeg'
date: '2019-11-01'
description: 'Gentle introduction to the concepts of GRPC with a practical implementation'
hideComments: false
keywords:
  - grpc
  - golang
  - python
readingTime: false
showFullContent: false
tags:
  - blog
  - programming

title: Understanding gRPC and implementing a practical application in Go and Python
---

If you’ve never heard of gRPC and want to know what it is, this post is for you. You will learn about a powerful new tool you can use for developing distributed applications.

gRPC is a complex topic, and my aim is to provide a gentle introduction and provide the intuition behind the concepts. We’ll first review the basics and then the concepts of gRPC. In the end, we will see a practical example in Golang and Python.

Before we dive into gRPC, let’s review what RPC is. If you’re already familiar with it, you can safely skip to gRPC. Let’s get started.

### What is RPC?

If you paid attention in your Distributed Systems class, you know what is RPC. But for those who didn’t attend those lectures, RPC stands for _‘__Remote Procedure Call’__._

RPC is an old concept. Since the 70s and 80s, this technique has been used in developing distributed systems. RPC is defined as follows:

> Remote Procedure Call is a protocol that one program can use to request a service from a program located in another computer on a network […]

Let’s break it down. It is a protocol — that means there are multiple ways it can be implemented in different environments. You will find an implementation of RPC in C++ and a different implementation for Java.

From the definition, we understand that RPC means that one system (client) is requesting some service from another system (server). But this definition doesn’t really explain why it’s called a _Remote Procedure Call._

In any programming language, you have the concept of a function. In technical terms, this is referred to as a  _procedure__._ Simply speaking, any function that you write in your program is a _procedure_ and anytime you invoke or call this function, you are doing a _procedure call_. So another way to look at RPC is one system is calling a _procedure_ (or function)  on another (remote) system.

Whenever you call a local procedure or invoke a function, the following things occur in order:

-   The current state of the process is pushed on stack so that it can be restored later.
-   The passed parameters for the function are gathered, and the process execution jumps to the place where the function is defined.
-   The function is executed using the passed parameters, and the result is gathered.
-   Then we go back to our previous state of execution while taking the result with us.

Imagine a function as a black box. You pass some input values and you get your output value. You are not concerned with all the steps that take place to get that result. For example:

int a = add(10, 14);

In RPC, things are pretty much the same. You pass input values and you get your output value at the point in your program. You give `10` and `14` to the black box and it will give you `24` in your result variable `a`.

The magic of RPC is that the actual addition of the numbers or the execution of the function does not happen on your machine when you call that function. What happens is something like this:

-   The current state of the process is pushed on stack so that it can be restored later.
-   The value of the passed parameters is gathered and a message packet is constructed which contains the function name and the parameter values.
-   This message  packet is sent to a remote system. The remote system unpacks this message, looks at the function name and the parameters. The remote system then executes that function with the passed values.
-   The remote system constructs a message which contains the result of the function. It is sent back to our system.
-   Our system unpacks the message and gets the result. The previous state of execution in our program is restored and we get the result of the function call.

![](/images/understanding-grpc/rpc-diagram.png)

All this stuff is actually abstracted for the programmer. All the programmer is concerned about is calling the function and getting the result.

Now there are a lot of details that need to be taken care of before this whole system works. But for now, if you can wrap your head around this concept of calling a function on a remote computer and getting the result in your program, then we’re good to go!

### What is gRPC?

RPC is a protocol. gRPC is an implementation of the RPC protocol. From the earlier section, you can make a few observations regarding the RPC protocol:

-   There needs to be a strong agreement between client and server regarding function and parameter definitions.
-   The programs on the client and server systems do not necessarily have to be written in the same language as long as definitions are consistent.
-   We need some way of communicating messages between two systems over a network.

Let’s see how gRPC addresses each point.

It is imperative for any RPC system to have consistent function and parameter definitions. Otherwise, clients will ask to execute the `add` function, and if the server does not have any function named `add`, the whole thing breaks down.

gRPC addresses this point of preserving consistency across the systems by maintaining a common configuration file. The server and client are created using this common configuration (or rather definition file) which defines the RPC functions, parameters, and the return values.

The next point is that the client or server do not need to be written in the same language. To make this possible, we need a language-agnostic way to define the configuration file so that it is interpreted the same in any language. So, gRPC came up with their own definition language called [protocol buffers](https://developers.google.com/protocol-buffers/)  as the Interface Definition Language (IDL) for describing functions and parameters.

In everyday web systems, whenever the server and client want to interact, they use JSON formatted messages. The client and server in any language can interpret these JSON messages. But JSON is not very efficient to parse for a computer even though it’s more human-readable. Computers parse binary far quicker than higher-level message formats. So to address the third point mentioned above, gRPC uses a new binary message format called [protobuf](https://developers.google.com/protocol-buffers/) (yes, protocol buffers and protobuf are the same thing). The advantage is that protobuf is a very efficient way to communicate for two systems. You can learn more about protobufs in the provided link and see why it is better than JSON.

Okay, enough theory! Let’s go through an example implementation.

### Example in Python and Go

As we saw, gRPC is language independent. You can have clients in one language and server in another. Today we’ll see how Python and Go can talk to each other using gRPC.

I have a requirement to clean some text and extract keywords. This particular task is quite easy to perform in Python which provides tools like `nltk` for working with texts. But the rest of my program is in Golang. And I don’t want to perform the same task in Golang. So I decided to make a service in Python which will take some text and return the keywords. And we’ll be calling this service from Golang as RPC.

Before working with gRPC, we need to install some dependencies. You need to install the [protobuf](https://developers.google.com/protocol-buffers/docs/downloads) compiler and gRPC plugins for [Golang](https://grpc.io/docs/tutorials/basic/go/) and [Python](https://pypi.org/project/grpcio-tools/). I’m assuming you already have Golang and Python installed on your systems.

When using gRPC, the first step is always to write the common definition file. Let’s write a definition file for our required service which will contain a function call which will perform our required task.

Here, we are putting our data in two messages — `Request` and `Response` (you can name them anything). We also define a `KeywordService` which contains a `rpc GetKeywords` which takes a `Request` input and returns a `Response`. Since we want to return a list of `Keywords` in the `Response`, we used `repeated string`. You can check the syntax of protocol buffers in detail but for now this much is enough.

Now the next step is, we need to generate language specific interfaces from this definition file. A language specific interface in Golang and Python can be generated using the tools provided by the gRPC.

Create two folders named `python` and `golang` in the directory where you saved the definition file as `nltk_service.proto` .

After running these commands, you will see that `golang` and `python` folders now contain some auto generated code. You should NEVER modify the generated code. We will import the generated code in our programs to build the client and server.

You can build both client and server on both sides. But for this example, I only need a Python server which provides a service and a Golang client which makes use of that service. So we’ll be implementing the server in Python and Client in Golang.

### **Python Server**

We first have to import the auto-generated code into our program. Next we have to implement the RPC method `GetKeywords`. The generated code has a Python class `KeywordsServiceServicer`. We want to extend this class with our own and override the `GetKeywords` method.

While overriding the `GetKeywords` function, we get access to the input parameter as defined in the definition file i.e `Request` which contains a `string text`. So, we can access the parameter value using that. We process the `text` received from the client using some logic, and then we return a `Response` object with `repeated Keyword`. Note that all these classes we are using (`Request`, `Response`, and `KeywordServiceServicer`) are imported from the auto-generated python files.

Now that we have implemented the server logic, we need to add some boilerplate to start the actual server :

Save the above file as `server.py` in the folder `python`. You can run this file to start a gRPC server on port 6000.

Our server is ready to serve! Let’s get cracking on the client.

### **Golang Client:**

First create `nltk_service` folder in our `golang` folder to work around Golang import requirements. Then move the `nltk_service.pb.go` in that folder.

Create a file `client.go` in the `golang` folder and initialize it as go module by running `go mod init golang`.

Then we need to import auto-generated code into our Golang client. The imported code will contain some functions that we will use to build our client.

You need to provide the address. Since we’re running both programs on the same system, we’re taking `127.0.0.1` using port `6000` which we defined in our Python server.

We can call the service’s `GetKeyword` function to execute the RPC. In the above code, this is done inside the `MyKeywords` function. It will return an instance of `Response` as per the proto definition file.

Now let’s finish the code by adding a `main` function to our Golang client :

What we are doing is getting some text from user in an infinite loop. Then we’re getting the keywords in the text from the `nltk` service which contacts the Python server and gets the result. Then we get the final result as a list of keywords passed to our Golang program.

Here’s the same in action:

Press enter or click to view image in full size

![](/images/understanding-grpc/python-go-grpc-demo.gif)

Testing Python Server

and Golang Client

That’s it! We have created a python gRPC server and a Golang client.

It is important to note that I have barely scratched the surface when it comes to using gRPC. There are a lot of things you need to understand and include before you start coding microservices with gRPC. For example, how to handle errors? How to handle multiple connections? How to do load balancing? How to do service discovery? How to communicate securely? And so on..

But hopefully you got some idea of what gRPC is and how it works. And now you’re ready to do some research on your own. Another thing — even while following along with this guide, you might run across errors because I have not given exhaustive steps to do everything. I suggest that you try to figure out why something breaks by doing some research. It will help cement your understanding and gain more insight into the topic.

All the code used in this post can be found on [Github](https://github.com/Cartmanishere/grpc-python-golang).

Thank you for reading!
