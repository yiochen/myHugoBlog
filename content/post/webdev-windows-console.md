+++
date = "2017-03-03T22:27:21-05:00"
title = "Web Dev on Windows, Really?"
draft = false
categories = []
tags = []
bigimg = ""
subtitle = "How I tamed my console"

+++
> In this post I will try to explain my endeavor to make front-end development possible on a Windows machine.

A lot of people hate to do front-end dev on Windows (or any other devs except game programming) from what I see. I am no different. At work, I use Ubuntu 16, and I am so accustomed to all the benefit and convenience Linux brings. Packages installed with `apt-get` and `npm` can just run immediately. Ssh works flawlessly. I can simply fire up Webpack dev server to preview my Angular site. I can add a whole bunch of aliases to `.bashrc` to make my life easier. But on Windows, things just fall apart.

I had couple attempts to make my Windows machines work for front-end dev including installing a dual boot linux system. The the stuff I want to achieve on a dev machines can boil down the three points:

-   I want to be able to use `git` to push and pull code from Github.
-   I want to install `npm` packages.
-   I want to run local server to serve my website for development.

Those seems very simple, and there are already solutions for all of them. But they just don't feel easy enough for me. The main problem, I think, is the lack of a integrated environment that does everything. 

In this post I will talk about some of the Windows command prompt alternatives that might make your development on Windows simpler. 

Disclaimer first, most of the applications are in active development. They might have changed a lot after I tried them or when you read this post. Secondly, this post expresses only my personal opinion. They solve my problems on Windows, while you might have other needs for a development environment on Windows.

If you want to skip ahead, you can just read the [TLDR](#tldr)
### Git-Bash
![Git bash on Windows](http://res.cloudinary.com/yiou-me/image/upload/post-web-dev-on-windows/Gitbash.jpg)  
When installed [git](https://git-scm.com/) on windows, it came with a [MINGW](http://www.mingw.org/) terminal called Git Bash. Git Bash is cool, at least it has color. It solve one of the main problems, I can version control my code now! The disadvantage is, it doesn't do much besides interacting with git.
### Bash on Windows
![Bash on Windows](http://res.cloudinary.com/yiou-me/image/upload/v1488648802/post-web-dev-on-windows/bash.png)  
This is a very promising project Microsoft and Canonical are working on. I am using it on Windows 10 anniversary update. It is mind blowing. I would say, when it become more mature, it will make Windows one of the most favorable platform for development. If you don't know what it is, make sure to check out this [overview](https://msdn.microsoft.com/en-us/commandline/wsl/about). Currently there is a [bug](https://github.com/Microsoft/BashOnWindows/issues/468) with network interfaces that makes it unable to run webpack dev server. But I [heard](https://github.com/Microsoft/BashOnWindows/issues/468#issuecomment-275516458) it will be fixed on Windows Creators Update, which is expected to launch by the end of 2017.  
![network interface error on Windows Bash](http://res.cloudinary.com/yiou-me/image/upload/v1488648802/post-web-dev-on-windows/bash_error.png)  
### Node.js
![node](http://res.cloudinary.com/yiou-me/image/upload/v1488648994/post-web-dev-on-windows/node.png)  
[Insert Node JS Meme here]  
When you installed [node](https://nodejs.org/en/download/) on windows, it also extends the windows native command prompt. After that, you will be able to use Node or npm in regular cmd. But as the native cmd, we still can't use ssh. But webpack dev server runs perfectly on Node.js.
### MinGW and Cygwin
Those are two linux-like environments on Windows. [MinGW](http://www.mingw.org/) represent Minialist GNU for windows. As the name suggests, it extracts out the most essential functunalities from a Linux system, including compilers for C, C++, etc. It's very minimal, I don't recommend it for web dev. [Cygwin](https://www.cygwin.com/) is a lot larger than MinGW. It contains more POSIX APIs. However the installation process is very complicated. You could choose which features to install and there are hundreds of them. For beginner to unix environment, like I was, this is not very friendly. 


It seems that I can do all the things with a combination of all the terminals above. Maybe this is the way many people do Web dev on windows. When you need to push code, you use Git-Bash. When you need to make a ssh connection, you use [PuTTY](http://www.putty.org/).  When you do system programming or making Windows application, you use MinGW, Cygwin, or the new Bash on Windows. When you need a web server, you use the Node extended cmd. As long as I keep at least on of each terminals open, I am good to go.

But I tried that, and the result is terrible. All the terminal looks very similar. I can't distinguish them with ease. I found that myself often typed into incorrect terminals by accident.

## TLDR
For the TLDR, here is what I do finally.   
I chose Windows native cmd, extended with Node, and Git. I used [openSSH](http://sshwindows.sourceforge.net/) for ssh client. And I used a very powerful console emulator called [conEmu](https://conemu.github.io/). After that, I can do everything I want in one terminal. More details see below. 

## Detail process
### How to run installed programs in cmd
If you know how to solve problems like 
```bash
'python' is not recognized as an internal or external command,
operable program or batch file.
```
You can skip this section.

This turned out to be one of the biggest problem I faced when I just started development on Windows. CMD CAN'T FIND ANYTHING!! Windows command prompt is not as smart as Linux terminal. When you start up cmd, it will search the paths defined in the environment variables. If you installed a new program, say python, the installer would probably add it to the path environment variables (If not, you have to add it yourself). Cmd doesn't know about it until it restarts and reloads the environment variables. Windows 10 made it much simpler for adding a path variable. Read more [here](http://www.computerhope.com/issues/ch000549.htm) if you need more information.

After the path is saved to the environment variable, make sure to **restart** command prompt. **This is very important**. 

### Install Node.js
If you do any web dev (especially Front-end), you probably need [NodeJS](https://nodejs.org/en/download/). I recommend installing the LTS (Long term support) version. Currently it is 6.10.0. Installing node will also install npm (currently 3.10.10). Note that, node updates a lot. By the time you read this post, it's probably many versions ahead already.

### Install Git
You still need to install [git](https://git-scm.com/) in your Windows. After that, you can use Git in cmd.

### Install openSSH
If you want to make ssh connection in cmd, you can use [openSSH](http://sshwindows.sourceforge.net/). You will need this if you want to clone from or push to Github using SSH. 



## Tabs in cmd?
So now our cmd is capable of doing a lot of things. But it is a still a ugly dark window. Actually, there are some awesome tools out there that we can use to make it more powerful.

Honorable mention first, [Hyper](https://hyper.is/) is a cross platform terminal built uisng Github [Electron](https://electron.atom.io/). It is highly customizable. You can connect it with various terminals including cmd, Powershell, etc. It supports splitting view (If you uses [Visual Studio Code](https://code.visualstudio.com/), you will know how convenient that is). But it is currently in early development stage, so it's not very stable.

![hyper](http://res.cloudinary.com/yiou-me/image/upload/v1488649280/post-web-dev-on-windows/hyper.png)  

I like [conEmu](https://conemu.github.io/) the best. It also supports tabs. You can set transparency for the background. You can customize the tab to display only the essential information (for me, I let it display current running process name and current folder name). It comes with a status bar in the bottom. It also has a [Quake mode](https://conemu.github.io/en/SettingsQuake.html), which makes the the terminal hide and show from the top of the screen, just like in the games. There are a lot more. Most of the settings have tooltips. So I would encourage you to play around with it.

![conEmu](https://conemu.github.io/img/ConEmu-Maximus5.png)  

That's it for this post. Like I said, I am not an expert. But this is how I created my development environment in a Windows 10 machine. Hope you enjoy it :)
