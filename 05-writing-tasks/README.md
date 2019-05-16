# Writing Tasks

> **Difficulty**: Basic

> **Time**: Approximately 15 minutes

In this exercise you will write your first Puppet Tasks for use with Bolt. 

- [How do tasks work?](#how-do-tasks-work)
- [Write your first task in Bash](#write-your-first-task-in-bash)
- [Write your first task in PowerShell](#write-your-first-task-in-powershell)
- [Write your first task in Python](#write-your-first-task-in-python)

# Prerequisites

Complete the following before you start this lesson:

1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory. 


# How does a Bolt-task work?

A [task](https://puppet.com/docs/bolt/latest/writing_tasks.html) allows a multi step configuration algorithm to be broken down into well defined steps. This philosphy is similar to breaking down a large function into smaller functions. Each smaller function is connected to other functions via variables. 

1. Tasks are scripts written in popular scripting languages like Bourne Shell, Powershell, Python, Ruby or it can be a Bolt-Task. You can download Bolt tasks from the [Puppet Forge](https://forge.puppet.com/).  Try this search: _sqlserver_ on the Forge and you will see it return a result with the Bolt Tasks icon next to it.

2. Tasks are similar to scripts, but they are kept in modules and can have metadata. This allows you to reuse and share them. Tasks are stored in the following hierarchy `modules/<unique directory name>/tasks/` underneath the Boltdir. 

3. You can have several tasks per module. 

4. The `init` task is special and runs by default if you do not specify a task name.

4. Tasks take arguments as environment variables prefixed with `PT` (short for Puppet Tasks). 

# Run your first task in Bash

This exercise uses `sh`, but you can use Perl, Python, Lua, or JavaScript or any language that can read environment variables or take content on stdin.

1. View the following file : `modules/exercise5/tasks/init.sh`. Note the $PT_`message` variable, where `message` is an input parameter. It will be given a value on the command line.

    ```
    #!/bin/sh
    
    echo $(hostname) received the message: $PT_message
    ```

2.  You run a task as: `bolt task run <task-name> <task options>`.

    Note the `message=hello` argument. This will be expanded to the `PT_message` environment variable expected by our task. By naming parameters explicitly it's easier for others to use your tasks.

    ```
    bolt task run exercise5 message=hello --nodes linux_nodes --modulepath ./modules
    ```
    The result:
    ```
    Started on localhost...
    Started on localhost...
  Finished on localhost:
    localhost.localdomain received the message: hello
    {
    }
  Finished on localhost:
    localhost.localdomain received the message: hello
    {
    }
  Successful on 2 nodes: localhost:2222,localhost:2200
  Ran on 2 nodes in 0.56 seconds
    ```

3. In the previous lab, note the `--modulepath ./modulues` argument you specified. Usually the modulepath goes into the `bolt.yaml` file. View your `bolt.yaml` file and confirm that `./modules` path is there. 

Run the following command (without the `--modulepath` option and relying on the `bolt.yaml` setting) and see that it produces the same results as in previous step.

  ```
  bolt task run exercise5 message=hello --nodes linux_nodes
  ```


3. Run the Bolt command with a different value for `message` and see how the output changes.


# Run your first task in PowerShell

If you're targeting Windows nodes then you might prefer to implement the task in PowerShell. 

1. View the following file : `modules/exercise5/tasks/print.ps1`

    ```powershell
    param ($message)
    Write-Output "$env:computername received the message: $message"
    ```

2. Run the exercise5 task. 

    ```
    bolt task run exercise5::print message="hello powershell" --nodes win_nodes
    ```
    The result:
    ```
    Started on localhost...
    Started on localhost...
  Finished on localhost:
    WINDOWS2016 received the message: hello powershell
    {
    }
  Finished on localhost:
    WINDOWS2016 received the message: hello powershell
    {
    }
  Successful on 2 nodes: localhost:55985,localhost:2202
  Ran on 2 nodes in 1.70 seconds
    ```

    **Note:**
    
    * The name of the file on disk (minus any file extension) translates to the name of the task when run via Bolt, in this case `print`.
    * The name of the module (directory) is also used to find the relevant task, in this case `exercise5`.
    * As with the Bash example above, name parameters so that they're more easily understood by users of the task.
    * By default tasks with a `.ps1` extension executed over WinRM use PowerShell standard agrument handling rather than being supplied as prefixed environment variables or via `stdin`. 

# Run your first task in Python

Note that Bolt assumes that the required runtime is already available on the target nodes. For the following examples to work, Python 2 or 3 must be installed on the target nodes. This task will also work on Windows system with Python 2 or 3 installed.

1. View the following : `modules/exercise5/tasks/gethost.py`

    ```python
    #!/usr/bin/env python
    
    import socket
    import sys
    import os
    import json
    
    host = os.environ.get('PT_host')
    result = { 'host': host }
    
    if host:
        result['ipaddr'] = socket.gethostbyname(host)
        result['hostname'] = socket.gethostname()
        # The _output key is special and used by bolt to display a human readable summary
        result['_output'] = "%s is available at %s on %s" % (host, result['ipaddr'], result['hostname'])
        print(json.dumps(result))
    else:
        # The _error key is special. Bolt will print the 'msg' in the error for the user.
        result['_error'] = { 'msg': 'No host argument passed', 'kind': 'exercise5/missing_parameter' }
        print(json.dumps(result))
        sys.exit(1)
    ```

2. Run the task on the `linux_nodes` inventory group

    ```
    bolt task run exercise5::gethost host=google.com --nodes linux_nodes
    ```
    The result:
    ```
    Started on node1...
    Finished on node1:
      google.com is available at 172.217.3.206 on localhost.localdomain
      {
        "host": "google.com",
        "hostname": "localhost.localdomain",
        "ipaddr": "172.217.3.206"
      }
    Successful on 1 node: node1
    Ran on 1 node in 0.97 seconds
    ```

# Extra points: Run the python task on the Windows nodes

1. We need to download the Python 3.7 interpreter on the `win_nodes` inventory group first.

  ```
  bolt command run "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri 'https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe' -Outfile 'c:\python-3.7.3.exe'"  -n win_nodes
  ```

2. Execute the installer to install Python 3.7 on the windows nodes

  ```
  bolt command run "cmd /K 'c:\python-3.7.3.exe InstallAllUsers=1 PrependPath=1 Include_test=0  /quiet'"  -n win_nodes
  ```

3. Verify python is installed 

  ```
  bolt command run "python -V"  -n win_nodes
  ```

4. Execute the python task on `win_nodes` inventory group.

  ```
  bolt task run exercise5::gethost host=google.com --nodes win_nodes
  ```
  Result:
  ```
  Started on localhost...
  Started on localhost...
Finished on localhost:
  google.com is available at 216.58.195.78 on windows2016
  {
    "host": "google.com",
    "ipaddr": "216.58.195.78",
    "hostname": "windows2016"
  }
Finished on localhost:
  google.com is available at 216.58.195.78 on windows2016
  {
    "host": "google.com",
    "ipaddr": "216.58.195.78",
    "hostname": "windows2016"
  }
Successful on 2 nodes: localhost:55985,localhost:2202
Ran on 2 nodes in 6.48 seconds
  ```

Once that command works, run the same Python task on both Linux & Windows nodes:

  ```
  bolt task run exercise5::gethost host=google.com --nodes win_nodes,linux_nodes
  ```
Result:
  ```
  Started on localhost...
  Started on localhost...
  Started on localhost...
  Started on localhost...
  Finished on localhost:
    google.com is available at 216.58.195.78 on localhost.localdomain
    {
      "host": "google.com",
      "hostname": "localhost.localdomain",
      "ipaddr": "216.58.195.78"
    }
  Finished on localhost:
    google.com is available at 216.58.195.78 on localhost.localdomain
    {
      "host": "google.com",
      "hostname": "localhost.localdomain",
      "ipaddr": "216.58.195.78"
    }
  Finished on localhost:
    google.com is available at 216.58.195.78 on windows2016
    {
      "host": "google.com",
      "ipaddr": "216.58.195.78",
      "hostname": "windows2016"
    }
  Finished on localhost:
    google.com is available at 216.58.195.78 on windows2016
    {
      "host": "google.com",
      "ipaddr": "216.58.195.78",
      "hostname": "windows2016"
    }
  Successful on 4 nodes: localhost:55985,localhost:2202,localhost:2222,localhost:2200
  Ran on 4 nodes in 6.18 seconds
  ```

# Next steps

Now that you know how to write tasks you can move on to:

[Downloading and running existing tasks](../06-downloading-and-running-existing-tasks)
