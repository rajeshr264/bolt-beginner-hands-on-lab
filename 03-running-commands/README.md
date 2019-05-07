# Running Commands

> **Difficulty**: Basic

> **Time**: Approximately 5 minutes

You can use Bolt to run arbitrary commands on a set of remote hosts. Let's see that in practice before we move on to more advanced features. Choose the exercise based on the operating system of your test nodes.

- [Running shell commands on Linux nodes](#running-shell-commands-on-linux-nodes)
- [Running PowerShell commands on Windows nodes](#running-powershell-commands-on-windows-nodes)

# Prerequisites


1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory.  

# Running shell commands on Linux nodes

Bolt by default uses SSH for transport. If you can connect to systems remotely, you can use Bolt to run shell commands.  

To run a command against a remote Linux node, use the following command syntax:
```
bolt command run <command> --nodes <nodes>
```

1. Run the `uptime` command to view how long the system has been running, with the _inventory group_ called `linux_nodes` in the `inventory.yaml` file.  
    ```
    bolt command run uptime --nodes linux_nodes
    ```
    The result:
    ```
    Started on localhost...
    Started on localhost...
    Finished on localhost:
      STDOUT:
        17:23:10 up  2:15,  0 users,  load average: 0.00, 0.01, 0.05
    Finished on localhost:
      STDOUT:
        17:23:10 up  2:16,  0 users,  load average: 0.00, 0.01, 0.04
    Successful on 2 nodes: localhost:2222,localhost:2200
    Ran on 2 nodes in 0.44 seconds
    ```

2. The `inventory.yaml` also allows you to specify the individual nodes in a comma-separated list.  You will the same result as before.

    ```
    bolt command run uptime --nodes linux-1,linux-2
    ```


# Running PowerShell commands on Windows nodes

Bolt can communicate over WinRM and execute PowerShell commands when running Windows nodes. 

1.  Run the following command to find the version Windows explorer executable installed on an inventory group called `win_nodes` in the `inventory.yaml`.  This command will be useful to check version of a product installed on a large number of windows machines. 

    ```
    bolt command run "Get-Process explorer -FileVersionInfo" --nodes win_nodes
    ```
    Result:
    ```
    Started on localhost...
    Started on localhost...
    Finished on localhost:
      STDOUT:
        ProductVersion   FileVersion      FileName
        --------------   -----------      --------
        10.0.14393.0     10.0.14393.0 ... C:\Windows\Explorer.EXE
    Finished on localhost:
      STDOUT:
        ProductVersion   FileVersion      FileName
        --------------   -----------      --------
        10.0.14393.0     10.0.14393.0 ... C:\Windows\Explorer.EXE
    Successful on 2 nodes: localhost:55985,localhost:2202
    Ran on 2 nodes in 0.82 seconds
    ```

# Next steps

Now that you know how to use Bolt to run _ad-hoc_ commands you can move on to:

[Running Scripts](../04-running-scripts)
