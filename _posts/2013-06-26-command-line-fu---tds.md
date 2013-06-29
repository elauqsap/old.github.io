---
layout: post
title: "Command Line Fu - TDS"
description: ""
category: command line fu
tags: [scripting, sys admin]
---
{% include JB/setup %}
About a year ago, I use to believe that I had a pretty good idea on how to use a Linux system from the command line. I realize now that I have barely scratched the surface of it's potential. So that is why one of my on going post segments will be about command line usage and scripting. To me, learning how to leverage the command line has been one of the most helpful techniques in bridging my gap into network security. Today I have a script that I wrote to deal with rogue processes on a system and I would like to walk through it with you.

You can check out the whole [project here](http://pasqualedagostino.github.io/tds) or you can follow along with the segments I provide. The script was written in bash and was done so for CentOS 6.3 so some of the commands used may have different syntax.

Let's start with ```set +x``` and ```set -x``` because this is an important debugging tool which allows you to see via the command line exactly what is happening as the code is executed. Just place the +x before and the -x after the bug and run the script to see the issue. 

{% highlight bash %}pts=`tty | awk -F/dev/ '{print $2}'`
ps -ef | grep ^$USER | grep -Ev "$pts( |$)" >> pidList
{% endhighlight %}

Now the following is key because it setups who we are so we don't kill our script or session. It does this by parsing our ```tty``` ( terminal attached to stdin/out ) value for our current psuedo terminal. The ```|``` character is what is known as a pipe and it connects the stdout of the previous command to the stdin of the next command which is awk. Awk is a pattern scanning and processing tool that can match on user defined fields and perform an action. In this case it is using ```/dev/``` as a field seperater and printing the second field which is our pts. The next line creates a list of PIDs ( Process IDs ) that do not stem from our subshell which we found with the previous command. The script will now grep ( regular expression tool ) through the ```ps -ef``` output for the current user and remove any lines that contain our pts. In the end we get a list and that list can be saved in a temporary file by using the redirection ```>>```. A good thing to know about redirection is ```>``` will create or erase and create a file with the redirected output. Where as ```>>``` while either create or append the output to the destination file. 

{% highlight bash %}for i in `cat pidList | awk '{print $2}'`;
do
	hup=$(ps -ux | awk '{print $2}' | grep $i)		
	if [ $? -eq 0 ];					
	then
		echo "Attempting sig HUP..."
		kill -HUP $i
		sleep 2s					
		term=$(ps -ux | awk '{print $2}' | grep $i)	
		if [ $? -eq 0 ]; 
{% endhighlight %}

This for loop is pretty interesting because for every line in the file it will perform the following evaluations. The ```cat pidList``` is the same as printing out the contents of the file to stdout then we are piping in that line by line output into the awk tool which is using it's default ( one space ) as the field separator. The variable ```i``` now has the value of the PID that awk just parsed out so now we are going to do a sanity check before moving on ( don't want to kill a non-existent process ). The sanity check is done by parsing the ```ps -ux``` command which lists processes by the current user and checking via grep for the PID stored in variable ```i```. The script does this sanity check by looking at the exit code ( ```$?``` ) to see if it is equal to 0 which means the previous command executed successfully. What this means for the script is that the process is still there and grep found it from that current PID list we just created. We can now proceed to stopping the process which our script has three ways to do so. The ```kill``` command has other signals but we are going to use HUP, TERM, and KILL to end a process with HUP being the most gracefully. From this point to the end of the for loop the script gets pretty redundant because all it is doing is performing the ```kill -OPTION $i``` command, sleeping, and then checking if the process survived. 

{% highlight bash %}
check=$(ps -ux | awk '{print $2}' | grep $i)	
if [ $? -eq 0 ];
then
	echo $i >> killFail	
	count=1					
fi;
{% endhighlight %}

At the end of each loop there is a check to see if the process survived. This code does a similiar check and then adds the PID to a file and then throws a flag that we later use for errors.

{% highlight bash %}
rm -rf pidList
if [ $count -eq 0 ];list
then
	echo "No processes survived!"
	exit 0
	else
		echo "Processes that survived:" >&2	
		for i in `cat killFail`;
		do
			cmd=`ps -ux | grep -v grep | grep $i | awk '{print $11,$12}'`
			pid=`ps -ux | grep -v grep | grep $i | awk '{print $2}'`
			stat=`ps -ux | grep -v grep | grep $i | awk '{print $8}'`
			echo -e "USER: $USER\tPID: $pid\tSTAT: $stat\t COMMAND: $cmd" 1>&2	
		done
	rm -rf killFail
	exit 1
fi;
{% endhighlight %}

The last bit here is to clean up and provide an exit code for the user as well as stderr. We can remove the files/directories the script made by doing ```rm -rf pidList``` and then we can proceed to checking the flag. If the flag does not get thrown then the script prints to stdout a success message and exits with a code of 0. If the flag is thrown then we cycle through the killFail log and build a list of processes that survived with some stats. This information is then sent to stdout but the ```>&2``` allows the user to redirect these error messages to stderr if they would like to.

**Example**
{% highlight bash %}
$ ./tds.sh 2> /path/to/tdsError.log
{% endhighlight %}

We remove the final file and exit with a code of 1 to show that not all of the processes we wanted to end were stopped properly. That is it for this script, please feel free to ask any questions you have and if I notice anything that might be helpful I will be sure to address it in the next CLF post.
