# Setting up test nodes and Bolt setup

## Bring up 4 test nodes using Vagrant

> **Time**: Approximately 20 minutes

In this exercise you will create nodes that you can use to experiment with Bolt. The attached Vagrantfile configures two CentOS-7 nodes and two Windows 2016 nodes.

1. Ensure you are in `bolt-beginner-hands-on-lab/02-acquiring-nodes` directory. 
1. Open the `Vagrantfile` in `02-acquiring-nodes` directory. It will show the 2 linux (_linux-1_ and _linux-2_) and 2 windows nodes (_win-1_ and _win-2_) being brought up.
1. Bring up the 4 test nodes:  `vagrant up --provider=virtualbox`
1. The previous step will take quite some time, especially downloading Windows 2016 Vagrant box. You will need a good internet connection.
1. Confirm the status of the 4 VMs are up using vagrant: `bolt-beginner-hands-on-lab/02-acquiring-nodes > vagrant status`

```
 Current machine states:
 linux-1                     running (virtualbox)
 linux-2                     running (virtualbox)
 win-1                       running (virtualbox)
 win-2                       running (virtualbox)
```
## Bolt setup

### Project Directory: Boltdir, creating a bolt.yaml file

Bolt runs in the context of a Bolt project directory called a *boltdir*. Any directory containing a __bolt.yaml__ file becomes a boltdir.

Note the __bolt-beginner-hands-on-lab/bolt.yaml__ file in the root directory of this lab. You will be _copying this bolt.yaml file to every lab_ that you will be doing.

### Building the inventory.yaml file

To connect to a bunch of machines, you can sometimes specify all the credentials, node names etc, on the Bolt command line. However, the better way is to specify all this information in an __inventory.yaml__ file. [Inventory File docs](https://puppet.com/docs/bolt/latest/inventory_file.html).

Note: Most of the keywords in the inventory file have a corresponding Bolt Command line option. Run `bolt help` to see all the command line options.

You will editing __bolt-beginner-hands-on-lab/inventory.yaml__ file and then _copying this inventory.yaml file to every lab_ that you will be doing.

#### Editing the Linux nodes part of the inventory.yaml file

1. `vagrant` has Linux node information that we need. Type:

    ```
    02-acquiring-nodes>  vagrant ssh-config > config.txt
    ```

2. Note the `Port` and `IdentityFile` values in this sample `config.txt` file :
```
Host linux-1
  HostName 127.0.0.1
  User vagrant
  Port 2222                 # Add this 'Port' value to linux-1 entry in 'inventory.yaml'
  PasswordAuthentication no
  IdentityFile /Users/foo/workshops/bolt-beginner-hands-on-lab/02-acquiring-nodes/.vagrant/machines/linux-1/virtualbox/private_key   # Add this 'IdentityFile' value to linux-1 entry in 'inventory.yaml'
  IdentitiesOnly yes
  LogLevel FATAL
  ForwardAgent yes

Host linux-2
  HostName 127.0.0.1
  User vagrant
  Port 2200                 # Add this 'Port' value to linux-2 entry in 'inventory.yaml'
  PasswordAuthentication no
  IdentityFile /Users/foo/workshops/bolt-beginner-hands-on-lab/02-acquiring-nodes/.vagrant/machines/linux-2/virtualbox/private_key     # Add this 'IdentityFile' value to linux-2 entry in 'inventory.yaml'
  IdentitiesOnly yes
  LogLevel FATAL
  ForwardAgent yes

Host win-1...
```

3. Open the `inventory.yaml` file in your favorite editor. My favorite editor is [Visual Studio Code](https://code.visualstudio.com/download)!

4. As shown above, locate the `Port` and `IdentityFile` values in the `config.txt` file .   

5. Add the value from the `Port` entry and replace it in `<add port number for linux-1 forwarded SSH port number 22>` in the `inventory.yaml` file.

6. Add the value from the `IdentityFile` entry and replace it in `<add IdentityFile value for linux-1 from config.txt file>` in the `inventory.yaml` file.

7. Repeat for `linux-2`.

#### Editing the Windows nodes part of the inventory.yaml file

1. Type: `vagrant port win-1`. 
2. From the output, note the line that shows how the WinRM port 5895 running in the Guest VM, is forwarded to the host (your laptop) machine.
3. Find this line :
```
win-1: 5895 (guest) => 55985 (host)
```
4. The forwarded port value i.e `55985`, is the value that goes into the `<add port number for win-1 forwarded WinRM port number 5895>` in the `inventory.yaml` file.
5. Repeat for `win-2` node.

# Sample inventory.yaml file

bolt-beginner-hands-on-lab/inventory.yaml 
```
groups:
  - name: linux_nodes
    groups:
      - name: linux-1
        nodes:
          - localhost:2200 # 'Port' value from linux-1 section, from config.txt
        config:
          ssh:
            user: vagrant
                             # 'IdentityFile' value to 'private-key', for linux-1 
            private-key: /Users/foo/bolt-beginner-hands-on-lab/02-acquiring-nodes/.vagrant/machines/linux-1/virtualbox/private_key 
            host-key-check: false
      - name: linux-2
        nodes:
          - localhost:2201 # 'Port' value from linux-2 section, from config.txt
        config:
          ssh:
            user: vagrant
                          # 'IdentityFile' value to 'private-key', for linux-2
            private-key: /Users/foo/bolt-beginner-hands-on-lab/02-acquiring-nodes/.vagrant/machines/linux-2/virtualbox/private_key # 'IdentityFile' value for linux-2 
            host-key-check: false
  - name: win_nodes
    groups:
      - name: win-1
        nodes:
          - localhost:55985  # From 'vagrant port win-1', forwarded port 5895 => 55985
      - name: win-2
          - localhost:2204   # From 'vagrant port win-2', forwarded port 5895 => 2204
    config:
      transport: winrm
      winrm:
        user: vagrant
        password: vagrant
        ssl: false              # NOTE: You must specify this 'ssl' as 'false' to use WinRM 
        extensions: [.py, .pl]  # Required for running Python or Perl scripts via WinRM
```
## Managing the nodes with Vagrant

You are now done with all the setup steps! You can chose to destroy all the VMs and bring them back up again. 

NOTE: When you launch them again, vagrant might change the `Port` and `IdentityFile` values. Verify, by running the `vagrant ssh-config` command.

### Destroy VMS
To delete all the VMs, type:
```
bolt-beginner-hands-on-lab/02-acquiring-nodes > vagrant destroy -f
```
or if you want to delete just the Windows VMs :
```
bolt-beginner-hands-on-lab/02-acquiring-nodes > vagrant destroy -f win-1 win-2
```

### Launch VMs again

To start all over again, type:
```
bolt-beginner-hands-on-lab/02-acquiring-nodes > vagrant up --provider=virtualbox
```
If you deleted just the Windows VMs, then only the Windows VMs will be started again. The Linux VMs will not be touched.
# Next steps

You can move on to next lab:

[Running Commands](../03-running-commands)
