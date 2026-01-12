---
author: Pranav Gajjewar
authorTwitter: ''
cover: /images/asynchronous-programming-cover.png
date: '2018-04-22'
dateFormat: '2006-01-02'
description: Explore python asynchronous programming via web scraping example
hideComments: false
keywords:
- python
- asynchronous
readingTime: false
showFullContent: false
tags:
- blog
- programming
title: Asynchronous Web Scraping in Python using concurrent module
---

Ever felt frustrated at how long your web scraping script takes to complete the task? Have you ever wished there was a faster way to do your web scraping?

Well, there is. And I’m going to show you today how you can increase the performance of your scraper in a very beginner friendly way.

In this post we will also talk about asynchronous programming in Python. And then apply that knowledge to optimize web scraping.

Let’s dive in!

## What is Asynchronous execution? And why would you want it?

If you’re a beginner in web scraping, then I assume you’ve worked with requests and BeautifulSoup modules in python. And what you generally do while writing your scraper is as follows —

```python
def parse(soup):
    # Extract data
    # return dataurls = [...]
    results = []

for url in urls:
    r = requests.get(url)
    soup = BeautifulSoup(r.content, 'lxml')
    results.append(parse(soup))
```

Or you might use a different structure than this. But the end result is same. The way you code your scraper is in a synchronous fashion.

What it means is that your program goes through the target URLs one by one, in a synchronized way. You send a GET request to the server and the server takes some time to send a response. But what do you suppose is happening while your program is waiting for a response from the server?

**Nothing!**

That’s right. The network request is the instruction that takes the most time in your script. And when you’re doing it in a synchronous way, your script remains idle a large amount of time which is spent waiting for the server response. How would you make use of that free time?

It’s quite obvious. We do not want our program to remain idle while one of the GET requests is waiting for server’s response. We want our program to move ahead with other URLS and their processing without being blocked due to one sluggish network request.

> Asynchronous programming is simply executing multiple instructions simultaneously.

So we need a way to process multiple URLs simultaneously and independent of one another. Let’s see how we can achieve this in Python.

## How asynchronous execution is achieved?

In this section, I will discuss different strategies of asynchronous execution. If you’re just interested in the asynchronous python code, you can skip this part.

There are many ways in which asynchronous execution is implemented. Three broad categories of multi-processing can be given as —

  - Process level multi-processing
  - Thread level multi-processing
  - Application level multi-processing

If you have some background in Unix operating system, you would be familiar with these concepts. Still, I will do my best to explain them as concisely and cogently as possible.

In Process level multi-processing, you can achieve asynchronous execution by dividing the total work across separate processes. Each process running on a different processor core. In this way, your original task is divided into number of chunks and all of these chunks are being processed simultaneously. This level of multi-processing is in-built in an OS. So all you have to do is utilize this and let the kernel worry about process scheduling.

Thread level multi-processing is almost same as the previous one. Except in this case, we are dividing the task across multiple threads. A thread is like a process but a lightweight process. And we can add multiple threads under a single process context. So all of these thread would belong to the same process. This feature is also implemented in the OS itself. We just need to utilize this using Python and we will see how it is done.

Application level multi-processing is somewhat different than the previous two. Here the OS is under the impression that it is executing only one process with a single thread. But our application itself schedules different tasks on that thread for execution. So the asynchronous nature of execution is implemented in our application program itself.

These are the main ways to handle parallel execution on a traditional Unix system. Now we will see how we can use the `concurrent` module in Python to utilize these concepts and to boost our scraping speed.

## Implementing asynchronous execution:

Okay, so you must be itching to get started. Let’s start coding —

```python
from concurrent.futures import ProcessPoolExecutor
from concurrent.futures import ThreadPoolExecutor
from concurrent.futures import Future
```

So we first import the things we require. You will observe that we imported `ProcessPoolExecutor` and `ThreadPoolExecutor`. Both of these classes correspond to Process level and Thread level multi-processing respectively. We only need to use one of these. And for our use case i.e web scraping, both of these will be effective.

So the multi-processing features in the OS are abstracted and we can directly do parallel processing using the above classes.

> The `concurrent.futures` module provides a high-level interface for asynchronously executing callables.
>
> The asynchronous execution can be performed with threads, using ThreadPoolExecutor, or separate processes, using ProcessPoolExecutor.

The way it works is that we have a pool of threads or processes. And we can assign some task to each of them and they will start executing independently of each other.

We can create a pool —

```python
pool = ThreadPoolExecutor(3) # This means a pool of 3 threads
#            OR
pool = ProcessPoolExecutor(3) # This means a pool of 3 processes
```

Now we can `submit` or `map` different tasks to each individual thread or process.

Suppose we have a list of 100 URLs and we want to download the HTML page for each URL and do some post-processing and extract data.

```python
def download_and_extract(url):
    r = requests.get(url)
    soup = BeautifulSoup(r.content, 'lxml')
    # Some data extraction logic
    return data
```

We have a function `download_and_extract` which will gather our data and we want to gather data from the 100 URLs previously mentioned.

If we were to do this synchronously, it would take 100 multiplied by average time for one GET request ( assuming post-processing time is trivial ). But instead if we divide the 100 URLs on 4 separate threads/processes, then the time required would be 1/4th the original time, at least theoretically.

So let us try this —

```python
URLs = [...]
def d_and_e(url): # Our download and extract function
    pass

with ProcessPoolExecutor(max_workers=4) as executor:
    futures = [ executor.submit(d_and_e, url) for url in URLs]
```

Here we have slightly modified the Pool initialization to suit our use case but it does the same thing when we initialized it previously.

`executor.submit` function takes two parameters in our code. The first one is the task we want to perform Or more technically, the function we want to execute and the parameters for the execution of our function. The executor will distribute the work across 4 different processes with each process executing one instance of `download_and_extract` for the given URL.

> (Future) Encapsulates the asynchronous execution of a callable.

This object represents the asynchronous execution of a specific function. You can read more about its properties in the [documentation](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Future). We will only focus on two main functions for this object that we will require viz. `done` and `result`.


`done()` function returns the bool value `True` if the function has finished executing or if there was some exception in it. And when it has finished execution, we can retrieve the result using the `result()` function.

```python
URLs = [...]
def d_and_e(url): # Our download and extract function
    pass

with ProcessPoolExecutor(max_workers=4) as executor:
    futures = [ executor.submit(d_and_e, url) for url in URLs ]
    results = []
    for result in concurrent.futures.as_completed(futures):
        results.append(result)
```

Here’s another new thing — `as_completed()`.

The function `as_completed()` simply determines the order of the results that are returned by the future. Using this function, we avoid having to write a block of code where we keep checking whether a given `Future` is `done()` or not.

The function will start generating results as soon as any one of the functions being executed yields some result. And then we simply append that result to our main collection of data.

And that’s it! Using these simple concepts you can make your program multi-processing capable. Web scraping is just a simple example to illustrate the concept. You can apply this concept anywhere you want.

We will look at a fully coded and working example below —

```python
import time
import requests
from bs4 import BeautifulSoup
from concurrent.futures import ProcessPoolExecutor, as_completed

URLs = [ ... ] # A long list of URLs.
def parse(url):
    r = requests.get(url)
    soup = BeautifulSoup(r.content, 'lxml')
    return soup.find_all('a')


with ProcessPoolExecutor(max_workers=4) as executor:
    start = time.time()
    futures = [ executor.submit(parse, url) for url in URLs ]
    results = []
    for result in as_completed(futures):
        results.append(result)
    end = time.time()
    print("Time Taken: {:.6f}s".format(end-start))
```

You can now experiment using this example with URLs of your choice and different degrees of parallelization. See what conclusions you can draw from this.

## What next?

In this post, I demonstrated how to divide a particular task across multiple threads and process. And we achieved asynchronous execution of a specific task in this way.

But think about this, the task we are doing i.e downloading data from the network, it is admittedly being done across multiple processes but on any one process the task is still being done synchronously.

What I mean is that we are simply performing the task in a parallel fashion. So in any one of the threads/processes, that one process or thread still remains idle for some time until server responds.

There is a way in which we can overcome this and make our scraping truly asynchronous. We would have to use Application level multi-processing to accomplish this.

We want our program to send a GET request and while the server is processing that request, we want our program to suspend that request and move on to next requests. When the server finally responds, we want that data to be mapped to the correct request. In this way, we do not allow our program to remain idle at all. It is always doing something.

This is possible in python using `asyncio` and `aiohttp` modules. I will explore both of those modules in the context of web scraping in a future post.
