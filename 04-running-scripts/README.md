# Running Scripts

> **Difficulty**: Basic

> **Time**: Approximately 5 minutes

In this exercise you will run existing *scripts* against remote nodes using Bolt.

- [Test Linux nodes for ShellShock](#test-linux-nodes-for-shellshock)
- [Test Windows external connectivity](#test-windows-external-connectivity)

# Prerequisites
Complete the following before you start this lesson:

1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory.  

# Test Linux nodes for ShellShock
Run the [bashcheck](https://github.com/hannob/bashcheck) script to check on ShellShock and related vulnerabilities.

**Tip:** You likely already have a set of scripts that you run to accomplish common systems administration tasks. Bolt makes it easy to reuse your scripts without modification and to run them quickly across a large number of nodes. Feel free to replace the bashcheck script in this exercise with one of your own. Just set the shebang line correctly and you can run scripts in Python, Ruby, Perl or another scripting language.

1. Run the `bashcheck` script in this directory on the `linux_nodes` inventory group. 

    ```
    bolt script run bashcheck --nodes linux_nodes
    ```
    The result:
    ```    
    Started on localhost...
    Started on localhost...
    Finished on localhost:
     STDOUT:
        Testing /usr/bin/bash ...
        Bash version 4.2.46(2)-release

        Variable function parser pre/suffixed [(), redhat], bugs not exploitable
        Not vulnerable to CVE-2014-6271 (original shellshock)
        Not vulnerable to CVE-2014-7169 (taviso bug)
        Not vulnerable to CVE-2014-7186 (redir_stack bug)
        Test for CVE-2014-7187 not reliable without address sanitizer
        Not vulnerable to CVE-2014-6277 (lcamtuf bug #1)
        Not vulnerable to CVE-2014-6278 (lcamtuf bug #2)
    Finished on localhost:
      STDOUT:
        Testing /usr/bin/bash ...
        Bash version 4.2.46(2)-release

        Variable function parser pre/suffixed [(), redhat], bugs not exploitable
        Not vulnerable to CVE-2014-6271 (original shellshock)
        Not vulnerable to CVE-2014-7169 (taviso bug)
        Not vulnerable to CVE-2014-7186 (redir_stack bug)
        Test for CVE-2014-7187 not reliable without address sanitizer
        Not vulnerable to CVE-2014-6277 (lcamtuf bug #1)
        Not vulnerable to CVE-2014-6278 (lcamtuf bug #2)
        Successful on 2 nodes: localhost:2222,localhost:2200
    Ran on 2 nodes in 0.51 seconds 
    ```

# Test Windows external connectivity
Create a simple PowerShell script to test connectivity to a known website.

**Tip:** You likely already have a set of scripts that you run to accomplish common systems administration tasks. Bolt makes it easy to reuse your scripts without modification and to run them quickly across a large number of nodes. Feel free to replace the script in this exercise with one of your own.

2. Run the `testconnection.ps1` script on the `win_nodes` inventory group.

    ```
    bolt script run testconnection.ps1 -n win_nodes
    ```
    The result:
    ```    
    Started on localhost...
    Started on localhost...
    Finished on localhost:
     STDOUT:
        Source        Destination     IPV4Address      IPV6Address                              Bytes    Time(ms)
        ------        -----------     -----------      -----------                              -----    --------
        WINDOWS2016   example.com     93.184.216.34                                             256      13
        WINDOWS2016   example.com     93.184.216.34                                             256      13
        WINDOWS2016   example.com     93.184.216.34                                             256      19
    Finished on localhost:
    STDOUT:
        Source        Destination     IPV4Address      IPV6Address                              Bytes    Time(ms)
        ------        -----------     -----------      -----------                              -----    --------
        WINDOWS2016   example.com     93.184.216.34                                             256      14
        WINDOWS2016   example.com     93.184.216.34                                             256      16
        WINDOWS2016   example.com     93.184.216.34                                             256      13

    Successful on 2 nodes: localhost:55985,localhost:2202
    Ran on 2 nodes in 5.92 seconds
    ```

# Next steps

Now that you know how to use Bolt to run existing scripts you can move on to:

[Writing Tasks](../05-writing-tasks)
