# Running Existing Tasks

> **Difficulty**: Intermediate

> **Time**: Approximately 10 minutes

In this exercise you will explore existing tasks, including several tasks that take advantage of Puppet under-the-hood.

- [Install Puppet using Bolt](#install-puppet-using-bolt)
- [The Tasks Playground](#more-tips-tricks-and-ideas-on-the-tasks-playground)

# Prerequisites

Complete the following before you start this lesson:

1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory. 

# Inspect installed tasks

Bolt is packaged with useful modules and task content.

- Run the 'bolt task show' command to view a list of the tasks installed in the previous exercise.

    ```
    bolt task show
    ```
    The result:
    ```    
    facts                              Gather system facts
    facts::bash                        Gather system facts using bash
    facts::powershell                  Gather system facts using powershell
    facts::ruby                        Gather system facts using ruby and facter
    package                            Manage and inspect the state of packages
    puppet_agent::install              Install the Puppet agent package
    puppet_agent::install_powershell
    puppet_agent::install_shell
    puppet_agent::version              Get the version of the Puppet agent package installed. Returns nothing if none present.
    puppet_agent::version_powershell
    puppet_agent::version_shell
    puppet_conf                        Inspect puppet agent configuration settings
    service                            Manage and inspect the state of services
    service::linux                     Manage the state of services (without a puppet agent)
    service::windows                   Manage the state of Windows services (without a puppet agent)

    Use bolt task show <task-name> to view details and parameters for a specific task.
    ```


# View and use parameters for a specific task

1. Run `bolt task show package` to view the parameters that the package task uses. 

    ```
    bolt task show package
    ```
    The result:
    ```    
    package - Manage and inspect the state of packages

    USAGE:
    bolt task run --nodes <node-name> package action=<value> package=<value> version=<value> provider=<value>

    PARAMETERS:
    - action: Enum[install, status, uninstall, upgrade]
        The operation (install, status, uninstall and upgrade) to perform on the package
    - name: String[1]
        The name of the package to be manipulated
    - version: Optional[String[1]]
        Version numbers must match the full version to install, including release if the provider uses a release moniker. Ranges or semver patterns are not accepted except for the gem package provider. For example, to install the bash package from the rpm bash-4.1.2-29.el6.x86_64.rpm, use the string '4.1.2-29.el6'.
    - provider: Optional[String[1]]
        The provider to use to manage or inspect the package, defaults to the system package manager

    MODULE:
    built-in module
    ```

2.  Using parameters for the package task, check on the status of the bash package:

    ```
    bolt task run package action=status package=bash --nodes linux_nodes
    ```
    The result:
    ```    
    Started on localhost...
    Started on localhost...
    Finished on localhost:
      {
        "status": "up to date",
        "version": "4.2.46-31.el7"
      }
    Finished on localhost:
      {
        "status": "up to date",
        "version": "4.2.46-31.el7"
      }
    Successful on 2 nodes: localhost:2222,localhost:2200
    Ran on 2 nodes in 9.08 seconds
    ```
3.  Using parameters for the package task, install the vim package across all your nodes:

    ```
    bolt task run package action=install package=vim --nodes linux_nodes --run-as root
    ```
    The result:
    ```
    Started on localhost...
    Started on localhost...
    Finished on localhost:
      {
        "status": "installed",
        "version": "2:7.4.160-5.el7"
      }
    Finished on localhost:
      {
        "status": "installed",
        "version": "2:7.4.160-5.el7"
      }
    Successful on 2 nodes: localhost:2222,localhost:2200
    ```

# Extra points: # Use the puppet_agent module to install puppet agent on linux nodes 

Install puppet agent with the install_agent task

``` 
bolt task run puppet_agent::install -n linux_nodes --run-as root
```
   
The result:
```
Started on localhost...
Started on localhost...
Finished on localhost:
  20:00:28 +0000 INFO: Version parameter not defined, assuming latest
  ...
Finished on localhost:
  20:00:28 +0000 INFO: Version parameter not defined, assuming latest
  20:00:28 +0000 INFO: Downloading Puppet latest for el...
  ...
Successful on 2 nodes: localhost:2222,localhost:2200
Ran on 2 nodes in 35.42 seconds
```

# More tips, tricks and ideas on the Tasks Playground

See the [installing modules](https://puppet.com/docs/bolt/latest/bolt_installing_modules.html) documentation to learn how to install external modules. 
These exercises introduce you to Bolt tasks. You'll find lots more tips, tricks, examples and hacks on the [Puppet Tasks Playground](https://github.com/puppetlabs/tasks-playground).

# Next steps

Now that you know how to run existing tasks with Bolt you can move on to:

[Writing Plans](../07-writing-plans)
