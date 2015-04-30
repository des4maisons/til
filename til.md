* Lita cannot have configuration in place for non-loaded plugins. In this example, I configured lita-hipchat without enabling:
* 
    ```
    [2015-04-30 17:55:51 UTC] FATAL: Lita configuration file could not be processed. The exception was:
    undefined method `hipchat' for #<Lita::Configuration:0x007ff6b12b3ea0>
    Full backtrace:kj
    ```
    
    On the other hand, certain plugins cannot be loaded without their configuration:
    ```
    [2015-04-30 18:02:28 UTC] FATAL: Configuration attribute "password" is required for "hipchat" adapter.
    ```
* [Lita](https://www.lita.io/) is a plugin-friendly ruby-based chat bot gem
* OSX puts its text dictionary in `/usr/share/dict/words`
* `knife` is a tool to change files in a chef-server at runtime. It can be run from a
    machine that isn't the chef server - it communicates updates to the server using an HTTP API. E.g.:

    ```bash
    knife node edit my-node01  # lets you edit the configuration for this node
    ```
* To run chef on a client machine, `sudo env -i chef-client`
