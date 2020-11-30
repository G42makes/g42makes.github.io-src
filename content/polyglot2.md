Title: Abusing Polyglot: Part 2
Date: 2020-11-29 12:00
Category: Python, Bash
Tags: python, bash, polyglot
Authors: Jared Young
Summary: Setting Bash Environment Variables From Python(kinda)
Status: draft

## Introduction
In Part 1, I discussed how I used a single file Polyglot script to ensure that the correct version of Python was used to execure the rest of the script.

A year or two later, one of my coworkers mentioned something about setting envronment variables from Python. This is generally deemed impossible, not just for Python, but for any program. A child process does not have access to modify the envronment of it's parent process. Usually the envronment that a child process sees is a full copy from the parent, so any changes do not stick around after the process ends. It's trivial for Python to set the envrionment for it's own child processes, and modify that as needed, but this is not about that process.

There are some ways around this limitation, but it's almost all cheating and not exactly what you expect. But if you need to create a regularly used Python(ish) script that sets local environment variables, keep reading.

## Starting with the Bash
There is one near workaround, and I was using it for some tools to save our users some time: you can execute your script in the same process as the current shell. This is super limited, in that it has to be written in Bash to run properly, and has to be called in the right way, but it does allow you to distribute a script that sets up environment variables.
If you happen to define functions within that Bash script, they become available for the user to call identically to any program, but still execute in the current process. 

The script has to either be called by the user directly(but properly) or included in the right way in the Bash profile, but once in place any functions can be called as if they were programs and those functions can set environment variables that persist in the current shell.

Calling the script matters, as if you run it like a normal bash script it will not work. I included some code to detect this in the script and it will let you know of the mistake and exit immediatly. If called with a '.' and a space before the name, Bash will execute it in the current process and everything will work properly. e.g. '. helper_functions.sh'.

## Adding in Python 
As mentioned in Part 1, it is relativly trivial to create a script that runs as Bash first and then Python once you are ready for it. And some of you may have realized how these two techniques can be combined... ish.

For the Python part, the script outputs valid Bash code. If you just execute the Python section with the right arguments, it should output on the console some Bash code that you could copy and paste to set the environment vars(or anything else really). 

For the Bash part, a set of functions are created that directly call the Python interpreter to run the rest of the script and then let Bash interpret the output.

Told you it was a hack. But it works. I don't use it in production, but in my tests I was able to set and modify any environment variable easily with code written in Python. The key reason I needed it was to simplify working with AWS' APIs, but there are plenty of other things you can do.

## Going Forward
This is already incredibly esoteric, and mostly just one of those things I use when having fun. I've not put this one into production, but I might if I were to re-write the code that inspired it, as the hoops I have to jump through for Bash to call the AWS CLI and other tools was a lot of work that I know is simpler in Python.

As for Part 3, what if I want to take this Polyglot structure and use it with another language. Maybe one that is not normally even executable directly most of the time anyway? Maybe install the language binaries before handing off to them? It's ugly, but I have a good bit of info on how to make this work with other tools.