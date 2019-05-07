# Writing Plans

> **Difficulty**: Intermediate

> **Time**: Approximately 10 minutes

In this exercise you will discover Puppet Plans and how to run them with Bolt. 

- [Write a plan using run_command](#write-a-plan-using-run_command)
- [Write a plan using run_task](#write-a-plan-using-run_task)

# Prerequisites
Complete the following before you start this lesson:

1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory. 

# How does a Bolt-Plan work?

Until now you have been running ad-hoc commands with Bolt. It was a mix of command line execution via bolt and ad-hoc commands, scripts and Bolt-task executions. 

Now, suppose you want to remove a node from a load balancer before you deploy the new version of the application, or to clear a cache after you re-index a search engine. This type of project will happen again and again. 

You need a way to codify _all the individual steps_.

Enter the Bolt-Plan. A *programmatic* way to capture a multi-step algorithm.

A Bolt-Plan is composed of multiple [Bolt-Plan Functions](https://puppet.com/docs/bolt/latest/plan_functions.html). 

# Using the 'run_command' plan-function in a Bolt-plan

This `run_command` function does the same job as the `bolt command run` CLI command.

Create a simple plan that runs a command on a list of nodes.

1. View the following : `modules/exercise7/plans/command.pp`:

    ```puppet
    plan exercise7::command (TargetSpec $nodes) {
      run_command("uptime", $nodes) 
    }
    ```

2. Run the plan:

    ```
    bolt plan run exercise7::command nodes=linux-1 --modulepath ./modules
    ```
    The result:
    ```    
    Starting: plan exercise7::command
    Starting: command 'uptime' on localhost:2222
    Finished: command 'uptime' with 0 failures in 0.37 sec
    Finished: plan exercise7::command in 0.38 sec
    Plan completed successfully with no result
    ```

    **Note:**

    * `nodes` is passed as an argument like any other, rather than a flag. This makes plans flexible when it comes to taking lists of different types of nodes or generating the list of nodes in code within the plan.

    * Use the `TargetSpec` type to denote nodes; it allows passing a single string describing a target URI or a comma-separated list of strings as supported by the `--nodes` argument to other commands. It also accepts an array of Targets, as resolved by calling the [`get_targets` method](https://puppet.com/docs/bolt/latest/writing_plans.html#calling-basic-plan-functions). You can iterate over Targets without needing to do your own string splitting, or as resolved from a group in an [inventory file](https://puppet.com/docs/bolt/latest/inventory_file.html).


# Using the 'run_task' plan-function in a Bolt-plan
Create a task and then create a plan that uses the task.

1. View the following task : `modules/exercise7/tasks/write.sh`. The task accepts a filename and some content and saves a file to `/tmp`.
    
    ```bash
    #!/bin/sh
    
    if [ -z "$PT_message" ]; then
      echo "Need to pass a message"
      exit 1
    fi
    
    if [ -z "$PT_filename" ]; then
      echo "Need to pass a filename"
      exit 1
    fi
    
    echo $PT_message > "/tmp/${PT_filename}"
    ```

2. Run the task directly with the following command:

    ```
    bolt task run exercise7::write filename=hello message=world --nodes=linux-1 --modulepath ./modules --debug
    ```
    
    **Note:**  It can be useful to trace the running of the task, and for that the `--debug` flag is useful. Here is the output when run with debug:
    
    ```
    Loaded inventory from /Users/foo/workshops/bolt-beginner-hands-on-lab/07-writing-plans/inventory.yaml
    Submitting analytics: {
      "v": 1,
      "cid": "530515db-86a1-4f5f-852a-e39f63b68197",
      "tid": "UA-120367942-1",
      "an": "bolt",
      "av": "1.13.0",
      "aip": true,
      "ul": "en-US",
      "cd1": "Darwin 18",
      "t": "screenview",
      "cd": "task_run",
      "cd5": "human",
      "cd4": 1,
      "cd2": 4,
      "cd3": 7
    }
    Completed analytics submission
    Loading modules from /opt/puppetlabs/bolt/lib/ruby/gems/2.5.0/gems/bolt-1.13.0/bolt-modules:/Users/foo/workshops/bolt-beginner-hands-on-lab/07-writing-plans/modules:/opt/puppetlabs/bolt/lib/ruby/gems/2.5.0/gems/bolt-1.13.0/modules
    Started with 10 max thread(s)
    Starting: task exercise7::write on localhost:2222
    Authentication method 'gssapi-with-mic' (Kerberos) is not available.
    Submitting analytics: {
      "v": 1,
      "cid": "530515db-86a1-4f5f-852a-e39f63b68197",
      "tid": "UA-120367942-1",
      "an": "bolt",
      "av": "1.13.0",
      "aip": true,
      "ul": "en-US",
      "cd1": "Darwin 18",
      "t": "event",
      "ec": "Transport",
      "ea": "initialize",
      "el": "ssh",
      "ev": 1
    }
    Running task exercise7::write with '{"filename"=>"hello", "message"=>"world", "_task"=>"exercise7::write"}' on ["localhost:2222"]
    Running task run '#<struct Bolt::Task name="exercise7::write", file=nil, files=[{"name"=>"write.sh", "path"=>"/Users/foo/workshops/bolt-beginner-hands-on-lab/07-writing-plans/modules/exercise7/tasks/write.sh"}], metadata={}>' on localhost:2222
    Started on localhost...
    Completed analytics submission
    Opened session
    Executing: mkdir -m 700 /tmp/73ffca0e-fe82-4cc7-bbbf-07f0d10d5ec6
    Command returned successfully
    Executing: chmod u\+x /tmp/73ffca0e-fe82-4cc7-bbbf-07f0d10d5ec6/write.sh
    Command returned successfully
    Executing: PT_filename=hello PT_message=world PT__task=exercise7::write /tmp/73ffca0e-fe82-4cc7-bbbf-07f0d10d5ec6/write.sh
    Command returned successfully
    Executing: rm -rf /tmp/73ffca0e-fe82-4cc7-bbbf-07f0d10d5ec6
    Command returned successfully
    Closed session
    Finished on localhost:

      {
      }
    {"node":"localhost:2222","status":"success","result":{"_output":""}}
    Finished: task exercise7::write with 0 failures in 0.49 sec
    Successful on 1 node: localhost:2222
    Ran on 1 node in 0.62 seconds
    ```
3. We are going to run this task using the `run_task` plan-function in the following plan: `modules/exercise7/plans/writeread.pp`.

    ```puppet
    plan exercise7::writeread (
      TargetSpec $nodes,
      String     $filename,
      String     $message = 'Hello',
    ) {
      
      # using the 'run_task' plan-function to execute the task. 
      run_task(
        'exercise7::write',
        $nodes,
        filename => $filename,
        message  => $message,
      )

      # use the 'run_command' plan-function to execute a shell command.
      run_command("cat /tmp/${filename}", $nodes)
    }
    ```

    **Note:**
    
    * This plan shows how a Bolt-Task and a shell command are used together (via `run_command` & `run_task` Bolt-plan functions), in a Bolt-plan. 
    * The plan takes three arguments, one of which (`message`) has a default value. We'll see shortly how Bolt uses that to validate user input.
    * First you run the `exercise7::write` task from above, setting the arguments for the task to the values passed to the plan. This writes out a file in the `/tmp` directory.
    * Then you run a command directly from the plan, in this case to output the content written to the file in the above task.

4. Run the Bolt-plan :
    
    ```
    bolt plan run exercise7::writeread filename=hello message=world nodes=linux-1 
    ```
    The result:
    ```
    Starting: plan exercise7::writeread
    Starting: task exercise7::write on localhost:2222
    Finished: task exercise7::write with 0 failures in 0.44 sec
    Starting: command 'cat /tmp/hello' on localhost:2222
    Finished: command 'cat /tmp/hello' with 0 failures in 0.23 sec
    Finished: plan exercise7::writeread in 0.69 sec
    Plan completed successfully with no result
    ```

    **Note:**
    
    * `message` is optional. If it's not passed it uses the default value from the plan.
    * When running multiple steps in a plan only the last step will generate output.


# Next steps

Now that you know how to create and run basic plans with Bolt you can move on to:

[Writing advanced Tasks](../08-writing-advanced-tasks)
