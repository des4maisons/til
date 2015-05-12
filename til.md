* To list files in an installed debian package, `dpkg -L $package_name`
* One way of creating ruby libraris: In your library base file you should require all the files
    defining things you want at top-level scope. E.g.:

    ```ruby
    # lib/my-library.rb
    require 'my-library/version'

    Dir[File.join(File.dirname(__FILE__), 'my-library', 'top-level', '*.rb')].each { |lib| require lib }
    ```
* You need to install the `ruby-build` plugin for `rbenv` if you want to be able to `rbenv install 2.1.4`
* UID mappings. A single user can own a range of userids, useful for user namespaces. In `/etc/subuid` (see `man subuid`), you   can see what range each user has access to.
* [A list of vagrant boxes, complete with their download url](http://www.vagrantbox.es/)
* In bash, if `~` is quoted, it doesn't get expanded to your home directory.

    ```
    $ file ~
    /Users/marguerite: directory
    $ file "~"
    ~: cannot open `~' (No such file or directory)
    ```
* `lsof -i tcp:3000` is a hopefully portable command for seeing what process is running on port `3000`.
    (The incantation of `netstat` I used previously doesn't work on OSX)
* To run redis as installed by brew on Mac OSX, `redis-server /usr/local/etc/redis.conf`
* To see the actual executable you are using while under rbenv shims magic, use `rbenv which gem`
* To search for which `chef` nodes have a certain role (e.g. `foo`), use `knife search node role:foo`
* To assign the output of a shell command to a variable in a Makefile, use the `shell` command:

    ```make
    DATE = $(shell date '+%Y-%m-%d')
    ```
* Lita cannot have configuration in place for non-loaded plugins. In this example, I configured lita-hipchat without enabling:

    ```
    [2015-04-30 17:55:51 UTC] FATAL: Lita configuration file could not be processed. The exception was:
    undefined method `hipchat' for #<Lita::Configuration:0x007ff6b12b3ea0>
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
