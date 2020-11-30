Title: Abusing Polyglot: Part 1
Date: 2020-11-29 11:00
Category: Python, Bash
Tags: python, bash, polyglot
Authors: Jared Young
Summary: Single File Python and Bash
Status: draft

## Introduction
A while back I had a problem: I needed a single file Python script that I could provide to my users, but their environments could vary wildly on what was installed. Python2 and Python3 were in use at the time, and depending on how the system was configured, either one could be the default Python.

I wanted to be sure that the script ran in the latest version of Python available, but also make sure that the version of Python used had the required modules installed(in this case a version of Boto). I did not have permission to modify the users environment when this script ran, as it was just a utility, so I needed to make sure I ran the right version of Python for their setup.

Now, I could probably just run whatever version of Python is default and have it check for the module, and call the same script in another version if the module was not available. But, I wanted to keep things a bit simpler. 

## What is a Polyglot Program
(Wikipedia)[https://en.wikipedia.org/wiki/Polyglot_(computing)] describes it well, but in short it is a program that can run in 2 or more programming languages without modification. I don't think that the solution I use below is in the truest sense Polyglot, but I like the word and it's the best I have right now.

## The Solution: Bash/Python Polyglot 
A bit of digging let me to a StackExchange discussion about (complex shebang)[https://unix.stackexchange.com/a/66242] lines. The person wanted to do more then just call an interpreter from the first line, but could not get what they wanted there. I've run into this before, as on some operating systems(OSX) you are limited on how many arguments you can use on the shebang line.

The solution, which is well described at (this old blog post)[http://softwareswirl.blogspot.com/2011/06/starting-python-script-intelligently.html], is to create a section at the top of the file that is treated by Python as a docstring, but is also valid Bash. Within the docstring section, any Bash script can be executed. After any work is done in Bash, Python is called using 'exec' to replace the Bash process with a Python one. It's elegant, simple and keeps you to one file for shareable scripts.

## Going Forward
This technique has stuck with me over the years. In Part 2 I will go over an odd application of this technique that I came up with after being (nerd sniped)[https://xkcd.com/356/] by a coworker about setting environment variables from Python(in the parent process, which is impossible BTW...).
