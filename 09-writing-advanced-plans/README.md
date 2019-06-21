# Writing advanced Plans

> **Difficulty**: Intermediate

> **Time**: Approximately 10 minutes

In this exercise you will further explore Puppet Plans:

- [Write a plan which uses input and output](#write-a-plan-which-uses-input-and-output)
- [Write a plan with custom Ruby functions](#write-a-plan-with-custom-ruby-functions)
- [Write a plan which handles errors](#write-a-plan-which-handles-errors)

# Prerequisites
Complete the following before you start this lesson:

1. [Setting up test nodes](../02-acquiring-nodes)
2. Verify ports and private-key values in valid `bolt-beginner-hands-on-lab/inventory.yaml` are valid.
3. Copy over the `bolt-beginner-hands-on-lab/bolt.yaml` and `bolt-beginner-hands-on-lab/inventory.yaml` files to this directory. 

# More information about the Bolt's Plan Language

The Bolt Plan language can also make use of [Puppet's built-in functions](https://puppet.com/docs/puppet/6.0/function.html), [data types](https://puppet.com/docs/puppet/6.0/lang_data.html) and the language can be extended with [custom functions implemented in Puppet or Ruby](https://puppet.com/docs/puppet/6.0/writing_custom_functions.html). These concepts will be demonstrated in the following examples.

# Write a plan which uses input and output

In the previous Bolt-Plan exercise, you ran tasks and commands within the context of a plan. Now you will create a task that captures the return values and uses those values in subsequent steps. The ability to use the output of a task as the input to another task allows for creating much more complex and powerful plans. Real-world uses for this might include:

* A plan that uses a task to check how long since a machine was last rebooted, and then runs another task to reboot the machine on nodes that have been up for more than a week.
* A plan that uses a task to identify the operating system of a machine and then run a different task on each different operating system.

1. View the task that prints a JSON structure with an `answer` key with a value of true or false. Save the task as `modules/exercise9/tasks/yesorno.py`.
    
    **Note:** JSON is used to structure the return value. 

    ```python
    #!/usr/bin/env python
    
    """
    This script returns a JSON string with a single key, answer which
    has a boolean value. It should flip between returning true and false
    at random
    """
    
    import json
    import random
    
    print(json.dumps({'answer': bool(random.getrandbits(1))}))
    ```

3. Create a plan and save it as `modules/exercise9/plans/yesorno.pp`:

    ```puppet
    # TargetSpec accepts a comma-separated list of nodes
    plan exercise9::yesorno (TargetSpec $nodes) {
      # Run the 'exercise9::yesorno' task on the nodes you specify.
      $results = run_task('exercise9::yesorno', $nodes)

      # Puppet uses immutable variables. That means we have to operate on data, in
      # this case a ResultSet containing a list of Results. Those are documented in
      # Bolt's docs, but effectively its a list of result data parsed from JSON
      # objects returned by the task. Select - "filter" - only the Result objects
      # where the task printed '{"answer": true}'.
      $answered_true = $results.filter |$result| { $result[answer] == true }

      # Result objects also include a reference to the target they came from. Get a
      # list of the targets that answered 'true'.
      $nodes_subset = $answered_true.map |$result| { $result.target }

      # Run the 'uptime' command on the list of targets that answered 'true'.
      run_command('uptime', $nodes_subset)
    }
    ```

    Data types used in this example: [TargetSpec](https://puppet.com/docs/bolt/latest/writing_plans.html#targetspec), [ResultSet and Result](https://puppet.com/docs/bolt/latest/writing_plans.html#concept-2722)
    Functions used in this example:  [run_task](https://puppet.com/docs/bolt/latest/plan_functions.html#run-task), [filter](https://puppet.com/docs/puppet/6.0/function.html#filter), [map](https://puppet.com/docs/puppet/6.0/function.html#map), [run_command](https://puppet.com/docs/bolt/latest/plan_functions.html#run-command)

4. Run the plan. 

    ```bash
    bolt plan run exercise9::yesorno nodes=linux_nodes --modulepath ./modules
    ```
    The result:
    ```
    Starting: plan exercise9::yesorno
    Starting: task exercise9::yesorno on localhost:2222, localhost:2200
    Finished: task exercise9::yesorno with 0 failures in 0.46 sec
    Starting: command 'uptime' on localhost:2222, localhost:2200
    Finished: command 'uptime' with 0 failures in 0.25 sec
    Finished: plan exercise9::yesorno in 0.74 sec
    Plan completed successfully with no result
    ```
    **Note:** Running the plan multiple times results in different output. As the return value of the task is random, the command runs on a different subset of nodes each time.

# Write a plan with custom Ruby functions

Bolt supports a powerful extension mechanism via Puppet functions. These are functions written in Puppet or Ruby that are accessible from within plans, and are in fact how many Bolt features are implemented. You can declare Puppet functions within a module and use them in your plans. Many existing Puppet functions, such as `length` from [puppetlabs-stdlib], can be used in plans. 

1. View the following file `modules/exercise9/plans/count_volumes.pp`:

    ```puppet
    plan exercise9::count_volumes (TargetSpec $nodes) {
      $result = run_command('df', $nodes)
      return $result.map |$r| {
        $line_count = $r['stdout'].split("\n").length - 1
        "${$r.target.name} has ${$line_count} volumes"
      }
    }
    ```

2. Run the plan.

    ```bash
    bolt plan run exercise9::count_volumes nodes=linux_nodes --modulepath ./modules
    ```
    The result:
    ```
    Starting: command 'df' on linux-1,linux-2
    Finished: command 'df' with 0 failures in 0.5 sec
    [
      "node1 has 7 volumes",
      "node2 has 7 volumes"
    ]
    ```

3. Write a function to list the unique volumes across your nodes and save the function as `modules/exercise9/lib/puppet/functions/unique.rb`. A helpful function for this would be `unique`, but [puppetlabs-stdlib] includes a Puppet 3-compatible version that can't be used. Not all Puppet functions can be used with Bolt.

    ```ruby
    Puppet::Functions.create_function(:unique) do
      dispatch :unique do
        param 'Array[Data]', :vals
      end
    
      def unique(vals)
        vals.uniq
      end
    end
    ```

4. View the plan `modules/exercise9/plans/unique_volumes.pp`. It shows a plan that collects the last column of each line output by `df` (except the header), and prints a list of unique mount points.

    ```puppet
    plan exercise9::unique_volumes (TargetSpec $nodes) {
      $result = run_command('df', $nodes)
      $volumes = $result.reduce([]) |$arr, $r| {
        $lines = $r['stdout'].split("\n")[1,-1]
        $volumes = $lines.map |$line| {
          $line.split(' ')[-1]
        }
        $arr + $volumes
      }
    
      return $volumes.unique
    }
    ```
5. Run the plan. 

    ```bash
    bolt plan run exercise9::unique_volumes nodes=linux_nodes --modulepath ./modules
    ```
    The result:
    ```
    Starting: plan exercise9::unique_volumes
    Starting: command 'df' on localhost:2222, localhost:2200
    Finished: command 'df' with 0 failures in 0.42 sec
    Finished: plan exercise9::unique_volumes in 0.46 sec
    [
      "/",
      "/dev",
      "/dev/shm",
      "/run",
      "/sys/fs/cgroup",
      "/run/user/1000"
    ]
    ```

For more information on writing custom functions, see [Puppet's custom function docs](https://puppet.com/docs/puppet/5.5/functions_basics.html).  

# Write a plan which handles errors

By default, any task or command that fails causes a plan to abort immediately. You must add error handling to a plan to prevent it from stopping this way. 

1. View the following plan  `modules/exercise9/plans/error.pp`. This plan runs a command that fails (`false`) and collects the result. It then uses the `ok` function to check if the command succeeded on every node, and prints a message based on that.

    ```puppet
    plan exercise9::error (TargetSpec $nodes) {
      $results = run_command('false', $nodes)
      if $results.ok {
        notice("The command succeeded")
      } else {
        notice("The command failed")
      }
    }
    ```


2. Run the plan. 

    ```bash
    bolt plan run exercise9::error nodes=all --modulepath ./modules
    ```
    The result:
    ```
    Starting: plan exercise9::error
    Starting: command 'false' on localhost:2222, localhost:2200
    Finished: command 'false' with 2 failures in 0.37 sec
    Finished: plan exercise9::error in 0.38 sec
    {
      "kind": "bolt/run-failure",
      "msg": "Plan aborted: run_command 'false' failed on 2 nodes",
      "details": {
        "action": "run_command",
        "object": "false",
        "result_set": [
          {
            "node": "localhost:2222",
            "status": "failure",
            "result": {
              "stdout": "",
              "stderr": "",
              "exit_code": 1,
              "_error": {
                "kind": "puppetlabs.tasks/command-error",
                "issue_code": "COMMAND_ERROR",
                "msg": "The command failed with exit code 1",
                "details": {
                  "exit_code": 1
                }
              }
            }
          },
          {
            "node": "localhost:2200",
            "status": "failure",
            "result": {
              "stdout": "",
              "stderr": "",
              "exit_code": 1,
              "_error": {
                "kind": "puppetlabs.tasks/command-error",
                "issue_code": "COMMAND_ERROR",
                "msg": "The command failed with exit code 1",
                "details": {
                  "exit_code": 1
                }
              }
            }
          }
        ]
      }
    }
    ```

    Because the plan stopped executing immediately after the `run_command()` failed, no message was returned.

3. View the following new plan  `modules/exercise9/plans/catch_error.pp`. To prevent the plan from stopping immediately on error it passes `_catch_errors => true` to `run_command`. This returns a `ResultSet` like normal, even if the command fails.
    
    ```puppet
    plan exercise9::catch_error (TargetSpec $nodes) {
      $results = run_command('false', $nodes, _catch_errors => true)
      if $results.ok {
        notice("The command succeeded")
      } else {
        notice("The command failed")
      }
    }
    ```

2. Run the plan and execute the `notice` statement.

    ```bash
    bolt plan run exercise9::catch_error nodes=linux_nodes --modulepath ./modules
    ```
    The result:
    ```
    Starting: plan exercise9::catch_error
    Starting: command 'false' on localhost:2222, localhost:2200
    Finished: command 'false' with 2 failures in 0.39 sec
    The command failed
    Finished: plan exercise9::catch_error in 0.4 sec
    Plan completed successfully with no result
    ```

**Tip:** You can pass the  `_catch_errors` to `run_command`, `run_task`, `run_script`, and `file_upload`.

# Next steps
Now that you have learned about writing advanced plans you can deploy an app with bolt! 

[Deploying and Application](../10-deploying-an-application)


[puppetlabs-stdlib]: https://github.com/puppetlabs/puppetlabs-stdlib
