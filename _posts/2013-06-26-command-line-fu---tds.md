---
layout: post
title: "Command Line Fu - TDS"
description: ""
category: command line fu
tags: [scripting, sys admin]
---
{% include JB/setup %}
About a year ago, I use to believe that I had a pretty good idea on how to use a Linux system from the command line. I realize now that I have barely scratched the surface of it's potential. So that is why one of my one going post segments will be about command line usage and scripting. To me, learning how to leverage the command line has been one of the most helpful techniques in bridging my gap into network security. Today I have a script that I wrote to deal with rogue processes on a system and I would like to walk through it with you.

You can check out the whole [project here](http://pasqualedagostino.github.io/tds) or you can follow along with the segments I provide.

Let's start with ```set +x``` and ```set -x``` because this is an important debugging tool which allows you to see via the command line exactly what is happening as the code is executed. Just place the +x before and the -x after the bug and run the script to see the issue. Now the following is key because it setups who we are so we don't kill our script or session. It does this by parsing our ```tty``` ( terminal attached to stdin/out ) value for our current psuedo terminal. The ```|``` character is what is known as a pipe and it connects the stdout of the previous command to the stdin of the next command which is awk. Awk is a pattern scanning and processing tool that can match on user defined fields and perform an action. In this case it is using ```/dev/``` as a field seperater and printing the second field with is our pts. The next line creates a list of PIDs ( Process IDs ) that do not stem from our subshell which we found with the previous command. The script will now grep ( regular expression tool ) through the ```ps -ef``` output for the current user and remove any  
```
pts=`tty | awk -F/dev/ '{print $2}'`
```

```
ps -ef | grep ^$USER | grep -Ev "$pts( |$)" >> pidList
```

