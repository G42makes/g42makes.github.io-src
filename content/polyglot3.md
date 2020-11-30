Title: Abusing Polyglot: Part 3
Date: 2020-11-29 13:00
Category: Python, Bash, Ansible
Tags: python, bash, polyglot, ansible
Authors: Jared Young
Summary: Building an Ansible Playbook that can Install Ansible First
Status: draft

## Introduction
In Part 1 I talked about building a Bash/Python polyglot script that allowed me to make sure the version of Python I called was able to run the rest of the script. Part 2 shows a crazy way to leverage this into a script that can use Python to set local environment variables, a problem that is harder then is sounds on first glance.

But the idea of integrating Bash into a script, or even a non-script, is not limited to Python. There are a number of languages that could take advantage of this ability to realy customize the calling of the script, and in my case I decided to explore the idea with Ansible.

## The Challenge
Ansible is a tool that I enjoy working with. In this case I was working to setup a test environment that I was going to rebuild quite regularly for a cluster environment. I wanted to write the config in Ansible to ensure I had an identical setup each time I ran it, and allow me to track what I changed in Git as I played with the config.

One problem: I still had to login to the system once it was setup(from an image) to install Ansible. In some environments, like AWS, I can easily script this, but I was running on some bare hardware in my home lab and didn't want to modify the image file for various reasons.

## Ansible Shebang
Ansible playbooks are generally not considered scripts, but with the proper design of the playbook, you can make them executable. 

There are 3 key parts:
1. Put a shebang at the top of the script to allow your OS to know what program to execute the playbook: '#!/usr/bin/env ansible-playbook'
2. Ensure the host is 'localhost' and the connection is local.
3. Make the playbook executable: 'chmod a+x playbook.yml'

From there, you can just call the playbook directly, './playbook.yml' and watch it execute.

## Adding in a touch of Bash 
As seen in Part 1, you can sometimes hide a Bash script in another element of a different scripting language. In that case we used the docstring, but we don't have multi-line comments in YAML(the langage of the Ansible playbooks). Where we do have multi-line is with certain strings. All we need to do is make a task that the playbook will ignore, fill it with Bash code and make sure Bash can find it.

So, we create a section with 'hosts: none' to ensure it matches no systems, put a debug task in there where the msg is a multi-line YAML string and drop in some Bash commands. The multi-line string has to be formatted just right to start, and the last line should be an 'exit' to ensure Bash skips the rest of the file.

With a normal bash shebang at the top, '#!/usr/bin/env bash', the interpreter will try to run each line, dropping some errors on the way(TODO: try to supress the command not found errors). Once it finds the actual script, it will execute as normal and exit when done. If you happen to call the ansible-playbook command and reference the script, it will run quite happily.

# Ansible installing Ansible 
A bit of a chicken and the egg problem: you cannot install Ansbile on a system using Ansible unless it's already installed. With this technique, it's trivial to make a single executable script file that installs Ansible however you like, and executes the playbook from there.