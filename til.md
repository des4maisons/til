* If you mount a bundler project into a VM, make running bundler faster by
    installing the bundler dependencies in another directory on the VM.
* In `rspec`, to get a non-strict double (one doesn't care what method you
    call) that returns a non-strict double for any method you call on it, use
   ```ruby
   double().as_null_object
   ```
* `man proc` shows you all the things you can find under `/proc`
* rspec to halt after first failing test:
    ```
    rspec --fail-fast
    ```
* Gem project boostrapping:
    * Create the scaffolding for a gem:
        ```
        bundle gem gemname
        ```
    * Run `rspec --init` to generate all the `rspec` bootstrapping code you need
    * Adding bundler tasks to your `Rakefile` allows you to to run `rake build`
        to build your gem:
        ```
        Bundler::GemHelper.install_tasks
        ```
* `dig +short` will give you a terse answer to a `dig` query
* In tmux, if you `C-B` and immediately hit the up arrow key multiple times,
    you will cycle through the window panes.
* in mysql, if you have a `''@'localhost'` user, a `'marguerite'@'%'` user and
    NO `'marguerite'@'localhost'` user, then the `''@'localhost'` will take
    precedence. You might get permission denied if you try to use mysql on
    localhost using the `marguerite` user with `marguerite`'s password. If
    neither `''` or `'marguerite'` has a password, you might get `''`'s grants.
    Basically, you need to greate a `'marguerite'@'localhost'` user with
    `'marguerite'`'s password for `marguerite` to be able to log in on localhost
* When `nc -tzv` gives you multiple conflicting responses, e.g. as follows:
    ```
    nc: connectx to localhost port 3306 (tcp) failed: Connection refused
    found 0 associations
    found 1 connections:
         1: flags=82<CONNECTED,PREFERRED>
            outif lo0
            src 127.0.0.1 port 50410
            dst 127.0.0.1 port 3306
            rank info not available
            TCP aux info available

    Connection to localhost port 3306 [tcp/mysql] succeeded!
    ```
    it might be because you have 2 entries for `localhost` in your `/etc/hosts`
    \- one for ipv4 and one for ipv6. One of them is failing and the other is
    succeeding.
* In ruby documentation, `Klass.method` is referring to the class method
    `method` on `Klass`, and `Klass#method` is referring to the install method
    `method` on `Klass`.
* In GoCD, if you "rebuild" a failed build from the page containing the failed
    build, it will re-use the same config and everything (I think). So if you
    change the config (eg, you change the directory that a material should be
    checked out in), those changes will not affect the rebuilding of that build.
    BUT if you "rebuild" from the pipelines list page using the play button, it
    will apply the new config.
* Print public key associated with private key:
   ```
   ssh-keygen -y -f <priv-key-path>
   ```
* Multi-line comments in ruby:
    ```ruby
    =begin
    ^ should be at the beginning of the line
    This entire thing is a comment!
    =end
    ```
* In bash, `seq 1 1 15` will print numbers 1 to 15.
* How to break up long strings in ruby:
    ```ruby
    x = 'this '\
      'is a kinda '\
      'long string'
    ```
* `--include` and `--exclude` for `s3cmd sync` does not work the way I expect.
    Would have thought that `--exclude '*' --include '*/foo/console.log'` would
    pick up all `foo/console.log`. But in actuality I first need to `include`
    whatever directories `foo` might happen to be in.  At least that was my
    observation today.
* The environment variable `BUNDLER_GEMFILE` tells bundler which Gemfile to
    use, so you don't need to `cd` somewhere to run `bundle exec`.
* It is in theory easy to cross-compile pure go programs. From [this
    page](http://solovyov.net/en/2012/cross-compiling-go/):
    > I’m on OS X and my Go source code resides in ~/var/go. In this case what
        I do is:
        ```
        cd ~/var/go/src
        CGO_ENABLED=0 GOOS=linux GOARCH=amd64 ./make.bash
        CGO_ENABLED=0 GOOS=windows GOARCH=amd64 ./make.bash
        ```
        And afterwards in different directory:
        ```
        CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o gr-linux64 goreplace
        CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o gr-win64 goreplace
        ```
* To manage a bash script with an infinite loop:
    ```
    echo $$ > /var/run/my_script.pid
    ```
    and then, somewhere else, `kill $(cat /var/run/my_script.pid)`
* Attribute nesting in chef:
    > `ohai foo/bar`
    > `search foo_bar`
    > `knife foo.bar`
    > these are the three forms for specifying nesting..
* The chef `search` function uses `solr` as a backend. So you can't use `.` to
    describe nested attributes, because the `.` is already used for something
    in solr
* `jar -tf` for listing the contents of a jar
* `1>&2` at the end of your line to direct stdout to stderr. STOP FORGETTING!
* In bash, to use `<` to compare strings, you should use `[[`, and not `[`:
    ```
    $ [ a > b ]   # exits 0
    $ [[ a > b ]] # exists 1
    ```
* In iTerm, command+/ shows you where your cursor is
* Debugging a java program:
    - Check which jdk you are using. OpenJDK vs Oracle can behave differently
        when advanced java functionality is used
* "Moonlighting" means working at a secondary job. Historically: (in
    19th-century Ireland) the carrying out of cattle-maiming, murders, etc,
    during the night in protest against the land-tenure system
* ETL stands for "Extract, Transform, Load" - the process of taking data from
    sources, transforming it according to your needs, and storing it somewhere
    else.
* "Strategy" is a general plan for how you will acheive something. "Tactics"
    refers to the specifics of the many activities you may need to do to
    execute that strategy. Strategy happens before battle, tactics are
    implemented during.
* Bind mounting lets you mount the same filesystem at 2 different spots
* In ruby, you can do a set-like difference of arrays like this:
    ```ruby
    x = [1, 2, 3]
    y = [3, 4, 5]
    x - y
    ==> [1, 2]
    ```
    There is also the Set class, if you're into that sort of thing.
* In vim, `:ls` lists your buffers. `:b<n>` takes you to buffer `<n>`.
* The Net::HTTP lib lets you make web requests in ruby
* Show total amount of free memory: `free -m` (`-m` is for "megabytes")
* Some cheap ways to truncate a file:
    * `truncate -s 0 $dest_file`
    * `cp /dev/null $dest_file`
* To test this:

    ```ruby
    not_if { ::File.exists?("/usr/local/bin/jq") }
    ```

    You need to allow the `File` class to behave normally before you stub it.
    Chef and RSpec both have calls to `File.exist?`, so you need to only stub
    yours:

    ```ruby
    it "whatever" do
      allow(File).to receive(:exist?)
        .and_call_original
      allow(File).to receive(:exist?)
        .with("your/file/that/is/being/tested")
        .and_return(false)
      expect(chef_run).to run_bash('stuff')
    end
    ```
* SOP means "standard operation procedure"
* By default, `berks` (the `chef` equivalent of `bundler`) looks to
    http://cookbooks.opscode.com/ to download cookbooks listed in `Berskfile`.
* To remove the "Apps" button from your chrome bookmark bar: right click on the
    bar and click "Show Apps Shortcut"
* You can view the contents of an `ar`. `ar t` to list contents, `ar x` to
    extract.
* Debian packages are `ar` archive files.
* To test a given file in `rspec`:
    ```
    rake spec SPEC=/path/to/spec_file.rb
    ```
* In ChefSpec, notifications don't fire, since ChefSpec is attempting to stub
    out any actions that are taken as a result of a resource. This means you
    can't test something with `action :nothing`, since it won't get triggered.
* [vagrant share](https://docs.vagrantup.com/v2/share/index.html) exposes your
    vagrant ports to the outside internet.
* [`ngrok`](https://ngrok.com/) is a tool that sets up a tunnel from the
    outside internet to your computer.
* There's an anchor button on 1password menu-bar-drop-down-windows that lets
    you anchor the window on top of everything even after the 1password menu has
    been dismissed. Useful for when you know you're going to need that same
    password again and again.
* In `rspec` tests, `context` is the same as `describe`, and `describe` is a
    thing that lets you apply `before` and `let` blocks to a group of tests.
    You can nest `describes`, but it's easier to read if you use `context`
    inside of a `describe`.
* `-y` option for `strace`: displays full path in file descriptors. e.g.:
    ```
    [pid 28968] read(244</var/go/x.properties>, "prop1=stuff\nprop2=stompy"..., 8192) = 72
    ```
* Why `let` is better than instance variables in `before` blocks in `rspec`
    tests (See [this stack overflow answer](http://stackoverflow.com/questions/5359558/when-to-use-rspec-let/5359979#53599790)
* Authentication problems? Is it because you have 2-factor authentication
    enabled?
* Determining if your code is correct:
    1. isolate it to the simplest thing
    1. Get this simple thing under test
    1. review all booleans (should your trues be falses?)
    1. Did you spell anything wrong?
    1. Did you read the error message carefully?
* Troubleshooting/debugging problems of any kind:
    1. Is solving this problem necessary? Look bigger picture. Is the bigger
        thing you're trying to solve the right approach? Are you picking the
        wrong battles?
    1. Is this going to be yaks? Maybe there's a different problem you can solve.
    1. How long have you been fighting this? Is it time to ask for help?
* `OH` in twitter speak means "overheard"
* If you are debugging a network connection to a container, verify
    * that the ip that is being connected to is the ip of the machine you want
      to connect to
    * that in any web requests, you put the port in the right spot
    * that both the vm and the container are running
    * that iptable rules are not blocking/dropping any of your traffic
    * that traffic is going from the vm to the container as expected (tcpdump)
    * that nginx isn't running anywhere to mess up what you think should happen
    * that you aren't forwarding ports to any restricted ports such as 80 or 443
* `iptables -t nat -S <chain>` displays the rules for chain `<chain>`
* `iptables -n -L -v` displays numeric ip addresses and ports, lists all rules
  for (all) chains, and outputs the hit counts for each rule.
* `tcpdump tcp -i any -nn port xxxx` dumps any traffic on any interface, to or
  from port xxxx, and does not attempt to resolve port numbers or ip addresses.
* Most likely, knife is already configured on your chef server.
* There are 2 ways to bootstrap a new knife workstation. One involves having
  access to the chef server, the other requires the admin's pem file (key).
  Both involve having the validator.pem available on the workstation.
    * Using the admin pem:
    ```
    $ knife configure --initial
    WARNING: No knife configuration file found
    Where should I put the config file? [/home/jtimberman/.chef/knife.rb]
    Please enter the chef server URL: [http://chef.example.com:4000] https://chef.example.com
    Please enter a name for the new user: [jtimberman]
    Please enter the existing admin name: [admin]
    Please enter the location of the existing admin's private key: [/etc/chef/admin.pem] .chef/admin.pem
    Please enter the validation clientname: [chef-validator]
    Please enter the location of the validation key: [/etc/chef/validation.pem] .chef/chef-validator.pem
    Please enter the path to a chef repository (or leave blank):
    Creating initial API user...
    Please enter a password for the new user: Created user[jtimberman]
    Configuration file written to /home/jtimberman/.chef/knife.rb
    ```

    * using access to the server (you need to have configured knife there first):
    ```
    chef-server$ knife client create my-username -n -a -f /tmp/my-username.pem
    Created client[my-username]
    ----
    workstation$ mkdir ~/.chef
    workstation$ scp chef-server.com:/tmp/my-username.pem ~/.chef/my-username.pem
    workstation$ knife configure
    No knife configuration file found
    Where should I put the config file? [~/.chef/knife.rb]
    Please enter the chef server URL: [http://localhost:4000] http://chef-server.example.com:4000
    Please enter an existing username or clientname for the API: [my-username] my-username
    Please enter the validation clientname: [chef-validator]
    Please enter the location of the validation key: [/etc/chef/validation.pem] ~/.chef/validation.pem
    Please enter the path to a chef repository (or leave blank):
    WARN: You must place your client key in:
    WARN: /home/$USER/.chef/my-username.pem
    WARN: Before running commands with Knife!
    WARN:
    WARN: You must place your validation key in:
    WARN: /home/$USER/validation.pem
    WARN: Before generating instance data with Knife!
    WARN:
    WARN: Configuration file written to /home/$USER/.chef/knife.rb
    ```
* [This page on chef auth](https://docs.chef.io/auth.html) talks about
  how authorization/authentication to chef-server works (knife, chef-client...)
  including the bootstrapping process.
* If your keyboard is acting like it has a stuck key, but unsticking all keys
  doesn't fix it, nor does it work on your laptop, check that you don't have a
  3rd keyboard plugged in with something on top of it.
* Keep local SSH speedy when not connected to the internet: set `UseDNS`
  configuration to `no` in the ssh server config.
* [`sysdig`](http://www.sysdig.org/) is strace+tcpdump+lsof in one more
  user-friendly package.
* If your automation experiences a timeout during a `git clone` on `git
  ls-remote`, it's possible that `git` is expecting user input, such as a
  password to decrypt your private key (which in my case was empty!)
* An LWRP in chef is a Lightweight Resource or Provider. Chef provides 2 ways
  of implementing a resource (say). You can subclass a Chef::resource and write
  your own getters and setters, or you can define a resource using a simple DSL.
  [See this page](https://docs.chef.io/lwrp_custom.html)
* Vagrant plugins installed via `vagrant plugin install <name>` are published
  as gems in the official rubygems repo
* [goiardi](https://github.com/ctdk/goiardi) is an in-memory chef server
  written in go
* On OSX, `netstat` doesn't show you PIDs. You need to use `lsof` to see the
  PID:

    ```bash
    netstat -l | grep $SOME_IP   # grab the port. marguerite.<port_number>
    lsof -i tcp:$PORT_NUMBER   # here you can see the pid
    ```
* To list files in an installed debian package, `dpkg -L $package_name`
* One way of creating ruby libraris: In your library base file you should
  require all the files defining things you want at top-level scope. E.g.:

    ```ruby
    # lib/my-library.rb
    require 'my-library/version'

    Dir[File.join(File.dirname(__FILE__), 'my-library', 'top-level', '*.rb')].each { |lib| require lib }
    ```
* You need to install the `ruby-build` plugin for `rbenv` if you want to be
  able to `rbenv install 2.1.4`
* UID mappings. A single user can own a range of userids, useful for user
  namespaces. In `/etc/subuid` (see `man subuid`), you   can see what range
  each user has access to.
* [A list of vagrant boxes, complete with their download
  url](http://www.vagrantbox.es/)
* In bash, if `~` is quoted, it doesn't get expanded to your home directory.

    ```
    $ file ~
    /Users/marguerite: directory
    $ file "~"
    ~: cannot open `~' (No such file or directory)
    ```
* `lsof -i tcp:3000` is a hopefully portable command for seeing what process is
  running on port `3000`.  (The incantation of `netstat` I used previously
  doesn't work on OSX)
* To run redis as installed by brew on Mac OSX, `redis-server
  /usr/local/etc/redis.conf`
* To see the actual executable you are using while under rbenv shims magic, use
  `rbenv which gem`
* To search for which `chef` nodes have a certain role (e.g. `foo`), use `knife
  search node role:foo`
* To assign the output of a shell command to a variable in a Makefile, use the
  `shell` command:

    ```make
    DATE = $(shell date '+%Y-%m-%d')
    ```
* Lita cannot have configuration in place for non-loaded plugins. In this
  example, I configured lita-hipchat without enabling:

    ```
    [2015-04-30 17:55:51 UTC] FATAL: Lita configuration file could not be processed. The exception was:
    undefined method `hipchat' for #<Lita::Configuration:0x007ff6b12b3ea0>
    ```

    On the other hand, certain plugins cannot be loaded without their
    configuration:
    ```
    [2015-04-30 18:02:28 UTC] FATAL: Configuration attribute "password" is required for "hipchat" adapter.
    ```
* [Lita](https://www.lita.io/) is a plugin-friendly ruby-based chat bot gem
* OSX puts its text dictionary in `/usr/share/dict/words`
* `knife` is a tool to change files in a chef-server at runtime. It can be run
  from a machine that isn't the chef server - it communicates updates to the
  server using an HTTP API. E.g.:

    ```bash
    knife node edit my-node01  # lets you edit the configuration for this node
    ```
* To run chef on a client machine, `sudo env -i chef-client`
