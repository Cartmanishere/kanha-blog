---
author: Pranav Gajjewar
cover: ''
date: '2019-07-12'
dateFormat: '2006-01-02'
description: ''
hideComments: false
keywords:
- jupyter
- ssh
- jupyter-notebook
- remote
readingTime: false
showFullContent: false
tags:
- blog
- networking
title: Run jupyter notebooks on Remote Servers using SSH
---

Sometimes you want to run your code on a remote server. Sometimes your code is in the form of a jupyter notebook. But how do you run a jupyter notebook on a remote server and access the results in your local browser?

In this post, I will demonstrate how easy it is to accomplish  this.

In short, there are two ways to access Jupyter Notebooks running on remote server —

-   Using SSH Tunneling
-   Using Reverse Proxy

Now, if you’ve never heard of either of these, it is fine. That’s what this post is for!

Today, we are going to look at the former method. Using SSH is the recommended method since it is more secure and easier to setup. In some niche cases, you might have to go for the latter method. We will see that in another post.

### Access Server using SSH

As you may or may not know, when you connect to your remote server and you get the shell access, it is done through ssh. There are two ways general ways to authenticate with the remote server —

-   Using password
-   Using Pub/Priv Keys

Suppose you have a system on a local or remote network with IP `xx.xx.xx.xx` and you want to connect to it using ssh, the way you do it depends on whether you’re on windows or unix system.

On Unix, you might do something like `ssh username@xx.xx.xx.xx` and then you have to enter a password (password authentication) or you use your private key (which is the more secure method)

On Windows, you probably use [PuTTY](https://www.putty.org/) and there you have one of the two authentication methods setup.

The authentication  method is irrelevant. As long as you can connect to your system using SSH, you can run the jupyter notebook there and access it remotely. How? By using SSH Tunneling.

### What is SSH Tunneling?

This is also known as port forwarding. What this means is that you setup a network tunnel (a connection for data to flow) from a local point to remote point.

Example: `ssh username@``xx``.xx.xx.xx` `-NL 1234:localhost:1234` (for Unix)

Look  at the above example, `1234:localhost:1234` is the important part. What does it mean? It means that any network request you send to port `1234` in your current system will be automatically forwarded to `localhost:1234`  _from the remote system_.


![](/images/jupyter-notebooks-remote-server/ssh-tunneling-diagram.png)

You might have gotten some inkling on how it is possible to use SSH tunneling to access jupyter notebook from the above diagram. Let’s make it clear.

### Access Jupyter Notebook

Whenever you start jupyter notebook, you  can access it by going to `localhost:8888` in your browser. But that works because the jupyter is running on _your_ `localhost` . What about when the jupyter is running on someone else’s (remote server’s) `localhost` .

So what you need is a way  to connect to the remote server’s `localhost`from your local computer. That’s where SSH Tunneling comes into play.

Suppose your server’s IP is `172.26.36.128` as an example. You get shell access to your server using ssh (using any of your preferred method). Then you start the jupyter notebook —

jupyter notebook --no-browser --port 1234

This will start the jupyter  notebook on port `1234` . You will also get a URL with authentication token —

`http://localhost:8888/?token=8d186032bbbe095b294789e863b065a546fcc15b68683c99`

-   **O****n U****nix**
    To start SSH tunneling on unix, open your terminal and enter the following command —
    `s``sh` `-NL` `1234``:localhost:1234` `username@172.26.36.128``s``sh -NL 1234:localhost:1234` `-i` `/path/to/private_key`
-   **On Windows
    **You want to use PuTTY to start the SSH Tunnel, you can do it as follows —

![](/images/jupyter-notebooks-remote-server/putty-connection-details.jpeg)

Step 1: Enter the connection details

![](/images/jupyter-notebooks-remote-server/putty-tunnel-details.jpeg)

Step 2: The tunnel details

Now just open up your favorite browser and go to URL you got before and you will see the Jupyter Notebook running in your browser while the code in the notebook is executed on the remote server. That’s all!

### Conclusion

This is how you access jupyter notebook from a remote server. Some additional things you can do is run the SSH Tunnel command and Jupyter notebook command in the background. So you can have a jupyter notebook perpetually running on some dedicated port.

The above  method should satisfy the requirements of most people. But what if you want to access jupyter notebook without setting up any SSH Tunnel. You want to connect to it over the normal internet. In that case, you can setup a reverse proxy to access the jupyter notebook through a HTTP server on your remote machine. That method’s for another post :)

Stay tuned for next post!

Thank you for reading!
