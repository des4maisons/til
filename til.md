# JQ

* Display unix timestamps sensibly: ` | strftime("%Y-%m-%d")`
* To pretty print json with `jq`: `jq .`

# Package Management

* You can see what will be run when you install a deb with

    ```
    dpkg -I <package.deb>  # shows control file and other info
    ```
    or extract only the control information with

    ```
    dpkg -e <package.deb>
    ```
    You can do a dry run of installing with

    ```
    dpkg --dry-run -i <package.deb>
    ```
* Debs downloaded by apt are stored in `/var/cache/apt/archives`, unless someone
    cleaned up the system.
* Checksums for all the files in a deb are in
    `/var/lib/dpkg/info/<package-name>.md5sums`. You can use this to check for
    local modifications of a package-installed file.
* `aptitude why <package>` shows what packages depend on `<package>`.
* See what _installed_ package provides a certain file:

    ```
    dpkg --search <path-to-file>
    ```
* See what package provides a certain file:

    ```
    apt-get install apt-file
    apt-file update
    apt-file search <file>
    ```
* See installed packages on ubuntu:

    ```
    dpkg --get-selections | grep -v deinstall
    # or
    apt --installed list
    ```
* See all _explicitly_ installed packages:

    ```
    aptitude search '~i!~M'
    ```
* To extract a debian, `dpkg -x <debian> <target dir>`
* To list files in an installed debian package, `dpkg (-L|--listfiles)
    $package_name`
* To list files in a .deb file, `dpkg (-c|--contents) <file.deb>`
* To list files in a package in apt, `apt-file list <package name>`. Might have
    to `apt-get install apt-file`.
* To add an apt-repository,

    ```
    apt-get install -y software-properties-common python-software-properties
    add-apt-repository ppa:git-core/ppa
    curl -sSL '<url of key>' | apt-key add -
    apt-get update
    apt-get install -y git
    ```
* `pkgutil` allows you to query and manipulate Mac OSX Installer packages. eg:

    ```
    pkgutil --file-info `xcode-select -p` | grep pkg-version
    ```
* To install an apt package of a particular version, use

    ```
    apt-get install <package-name>=version
    ```
* To list all versions of apt packages available to install, use

    ```
    apt-cache madison <package-name>
    ```

    Don't ask me why though.
* You can view the contents of an `ar`. `ar t` to list contents, `ar x` to
    extract.
* Debian packages are `ar` archive files.

# SSL, SSH, scp

* To use wildcard globbing with `scp` on the remote server, quote the wildcard.

    ```
    scp 'servername:/path/to/dir/*' .
    ```
* To verify that an RSA priv/pub key pair match:

    ```
    openssl rsa -noout -modulus -in ~/junk/privk.txt | shasum
    ```
* To verify an RSA private/public key:

    ```
    openssl rsa -noout -in ~/junk/privk.txt
    openssl rsa -noout -pubin -in ~/junk/pk.txt
    ```
* If managing an ssh tunnel in bash, there are a few elegant ways to ensure it
    is closed in the end. These tips come from [this gist](https://gist.github.com/scy/6781836)
    * Pipe the command that needs the tunnel to the ssh command:

        ```
        mysql -e 'SHOW DATABASES;' -h 127.0.0.1 | ssh -L 3306:localhost:3306 remotehost cat
        ```

        We don't do this for the *purpose* of sending stdout to the remote
        server, but to ensure that when the calling program closes its stdout,
        the ssh connection will close.
    * Run `sleep` on the remote server

        ```
        ssh -f -o ExitOnForwardFailure=yes -L 3306:localhost:3306 sleep 10
        mysql -e 'SHOW DATABASES;' -h 127.0.0.1
        ```

        The connection will stay open as long as `mysql` is using it.
    * Tell ssh to use a control socket, which can later be used to control and
        shut down the connection.

        ```
        $ ssh -M -S my-ctrl-socket -fnNT -L 50000:localhost:3306 jm@sampledomain.com
        $ ssh -S my-ctrl-socket -O check jm@sampledomain.com
        Master running (pid=3517)
        $ ssh -S my-ctrl-socket -O exit jm@sampledomain.com
        Exit request sent.
        ```
* Even if I have already added a `known_hosts` entry for github, I still get
    these warnings when I `git clone`:

    ```
    Warning: Permanently added the RSA host key for IP address '192.30.252.130' to the list of known hosts.
    ```

    This is because of the ssh setting `CheckHostIp`, which verifies that the
    *ip* of the host is in `known_hosts`, in addition to the hostname. If the
    IP is not present in `known_hosts`, it is added, but that warning is issued.
* To remove a known-hosts entry,

    ```
    ssh-keygen -f ~/.ssh/known_hosts -R github.com
    ```
* To generate the fingerprint for a host's public key already in known-hosts,

    ```
    ssh-keygen -l -f ~/.ssh/known_hosts -F github.com
    ```
* To generate the fingerprint for an ssh public key,

    ```
    ssh-keygen -l -f /path/to/public_key
    ```
* On OSX, if you keep being told to type in your ssh key password while using
    `IdentitiesOnly yes` despite saving the password to your keychain, it is
    likely that you either have a public key file that is not in a format
    recognized by your OSX, or that you have no public key file. Generate a new
    one using `ssh-keygen -y -f id_rsa > id_rsa.pub`. Not great. `ssh
    -vvv` is not helpful in diagnosing this problem.
* On OSX, if you use `IdentitiesOnly yes` in your ssh config, you can still avoid
    typing in your password *if* you save your password to the system key chain.
    This will mean you will have to type in your password manually (no copy-paste)
    into the system dialog.
* `IdentitiesOnly yes` is how you tell ssh to only use the Identity you
    specify, and not rely on keys served by ssh-agent. My experience tells me
    that ssh-agent is not even used, so you will have to type in your password.
* `tar - | ssh "untar -"` vs `scp -r` for less back-and-forth over the network
    :)
* Print ssh public key associated with private key:

   ```
   ssh-keygen -y -f <priv-key-path>
   ```
* Print decrypted private key:

    ```
    openssl rsa –in enc.key -out dec.key
    ```
* Print rsa public key associated with a private key / pem file:

    ```
    openssl rsa -in key.pem -pubout
    ```
* Keep local SSH speedy when not connected to the internet: set `UseDNS`
  configuration to `no` in the ssh server config.

# iptables

* Instead of DROP or ACCEPT, you can LOG ip packets that match a filter:

    ```
    iptables -A FORWARD -s <source-ip> -d <dest-ip> -j DROP --log-prefix=XXX
    ```

    > When this option is set for a rule, the Linux kernel will print some
    > information on all  match‐ ing  packets  (like most IP header fields) via the
    > kernel log (where it can be read with dmesg or syslogd(8))
* In `iptables`, the *command* option short-forms tend to have capital letters:

    ```
    -P   --policy
    -A   --append
    -L   --list
    -h   < the exception!
    ```

    command *parameters* tend to be lower case:

    ```
    -j   --jump <target>
    -d   --destination <address>[/<mask>]
    -s   --source <address>[/<mask>]
    ```

    *non-command-parameter* options are also all lower case, and are limited to

    ```
    -v   --verbose
    -n   --numeric
    --line-numbers
    -x   --exact (packet, byte counters)
    -w   --wait (for the xtables lock)
    --modprobe=<command>
    ```
* `iptables -t nat -S <chain>` displays the rules for chain `<chain>`
* `iptables -n -L -v` displays numeric ip addresses and ports, lists all rules
    for (all) chains, number of refenences to those chains (if 0, then chain is
    not used), and outputs the hit counts for each rule.

# network, HTTP, DNS

* `tcpdump  -n icmp` to view traffic on icmp, do it from both sides of a connection to see who receives what
* Private address ranges:
    - 192.168.0.0 - 192.168.255.255
    - 172.16.0.0 - 172.31.255.255
    - 10.0.0.0 - 10.255.255.255
* To test UDP reachability/connectivity, use `nc -v -u -z -w <timeout>
    <dest-host> <port-range>`. This uses ICMP though which may be dropped.
* httpie: curl with features designed for humans!
* [This blog post](http://www.cyberciti.biz/faq/network-statistics-tools-rhel-centos-debian-linux/)
    has a treasure-trove of useful network diagnostic commands & tools.
* To see summary stats for each interface, such as dropped packets, `ip -s link`
* `netstat -s` will show a summary of stats for each protocol on the wire.
* `nstat` is a tool that shows you various stats about the network, such as
    number of tcp-resets, tcp retransmissions, new connections... Each run is
    relative to the last run. So if an event (such as a TCP Retransmit) hasn't
    happened since the last time you ran it, that statistic won't show up.
* To take a packet capture with tcpdump that can be read with wireshark,

    ```
    tcpdump [-i interface] -w packet_capture_file
    ```
* `iperf` measures actual maximum achievable bandwidth on IP networks. eg,
    - bandwidth
    - loss
    - mtu/mss
* In Wireshark, you can view all packets in the conversation of a particular
    packet by right clicking and clicking "follow".
* In Wireshark, you can view a tcp conversation flow by: Statistics > Flow Graph
* In wireshark, you can change the display of values like time and seq and ack
    to be *relative* to the start of the packet capture. This can help because
    it lets you see the time elapsed between / bytes increased by each requests.
* When analysing a packet-capture with wireshark, a sent ACK number of (say) X
    means an acknowledgement of all X - 1 bytes received so far, or looking at
    it another way, which byte number it's expecting next. This will correspond
    to the next SEQ number it expects to receive from the other server. See
    [this page](http://packetlife.net/blog/2010/jun/7/understanding-tcp-sequence-acknowledgment-numbers/).
* When analysing a packet-capture with wireshark, you can use a query language
    similar to:

    ```
    tcp.port eq <port> && tcp.seq eq <sequence_number>
    ```
* To rewrite a large capture file into many smaller ones,

    ```
    tcpdump -r <large_file> -w <new_file_prefix> -C <max_size_in_MB>
    ```
    This also works for sysdig.
* Clear OSX's dns cache:
    ```
    sudo killall -HUP mDNSResponder; sudo killall mDNSResponderHelper; sudo dscacheutil -flushcache
    ```

* Clear chrome's dns cache: go here
    [chrome://net-internals/#dns](chrome://net-internals/#dns) and push the button
* HTTP basic-auth via url:

    ```
    http://<username>:<password>@hostname
* In `/etc/hosts`, a line looks like this:

    ```
    IP_address canonical_hostname [aliases...]
    ```
    Note, `canonical_hostname` is not the same as `aliases`. This manifests
    itself when I run `chef-client` on a node. You can set the `node_name` of a
    node. However, It looks like `chef-client` will look up the
    `canonical_hostname` corresponding to that `node_name` in order to determine
    what client name it should use when authenticating with the chef server.
    E.g., the `node_name` should be `foo`, but I have this line in `/etc/hosts`:
    ```
    127.0.0.1 foo.bar foo
    ```
    and so `chef-client` presents the `node_name` as `foo.bar`.
    ```
* `dig +short` will give you a terse answer to a `dig` query
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
* [`ngrok`](https://ngrok.com/) is a tool that sets up a tunnel from the
    outside internet to your computer.
* If you are debugging a network connection, verify
    * that the ip that is being connected to is the ip of the machine you want
      to connect to
    * that in any web requests, you put the port in the right spot
    * that both the vm and the container are running
    * that iptable rules are not blocking/dropping any of your traffic. You can do this by
        * `itpables -Z` - this zeroes the rule hit counts in iptables
        * `watch iptables -n -L -v` to see up-to-date hit counts
        * send packets (via telnet or netcat) to you server, see if counts increase
        * Remember that `ssh` will constantly be sending packets as well
    * that traffic is arriving at hops along the way, including destination
        (tcpdump, below)
    * that nginx isn't running anywhere to mess up what you think should happen
    * that you aren't forwarding container ports to any restricted ports such
        as 80 or 443, if you are doing docker things
* `tcpdump tcp -i any -nn port xxxx` dumps any traffic on any interface, to or
  from port xxxx, and does not attempt to resolve port numbers or ip addresses.
* On OSX, `netstat` doesn't show you PIDs. You need to use `lsof` to see the
  PID:

    ```bash
    netstat -l | grep $SOME_IP   # grab the port. marguerite.<port_number>
    lsof -i tcp:$PORT_NUMBER   # here you can see the pid
    ```
* `lsof -i tcp:3000` is a hopefully portable command for seeing what process is
  running on port `3000`.  (The incantation of `netstat` I used previously
  doesn't work on OSX)

# docker, docker-compose, cgroups, lxc

* You can even avoid installing nsenter on the host by running it in docker:

    ```
    docker run -it --rm --privileged --pid=host debian nsenter
    ```
* [`nsenter`](https://manpages.debian.org/jessie/util-linux/nsenter.1.en.html)
    is a tool that lets you enter the namespace of a container while still
    having access to utilities installed on the host.
    Here is an [example of using tcpdump from the host inside the
    container](https://github.com/jpetazzo/nsenter#install-nsenter-docker-enter-and-importenv-into-the-vm)
* To gain console access to the docker-for-mac VM, use

    ```
    screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
    ```
* Sending `SIGUSR1` to Docker causes it to dump its stacktrace in the logs.
* If OOM killer is disabled for a memory cgroup and the cgroup runs out of
    memory, any access to that memory will hang, even if not in the memory
    cgroup. The process will become defunct.

    See [Reading /proc/pid/cmdline can hang forever](http://rachelbythebay.com/w/2014/10/27/ps/)
    and [this docker bug](https://github.com/docker/docker/issues/14440)
* You can get a bunch of diagnostic information about docker by running this:

    ```
    echo -e "GET /containers/6e4483df6241/stats HTTP/1.1\r\n" | nc -q 300 -U /var/run/docker.sock
    ```
    If you have a new enough version of curl, you can also just
    ```
    curl /var/run/docker.sock/containers/<containerid>/stats
    ```
    but older versions of curl don't support curling a socket.
* To control the *shutdown* timeout when you Ctrl-C a `docker-compose up`,
    you need to run `docker-compose up --timeout=20`
* Troubleshooting docker-compose networking:
    - Use `dc up` if you want ports forwarded. `run` doesn't, unless you
        specify the `--service-ports` option.
    - In your browser, use your docker-machine ip addr, not `127.0.0.1`.
    - Use `ports` not `expose` if you want ports forwarded to the host. Wrap
        `ports` entries in quotes to avoid yaml interpretation shenanigans.
        [Reference](https://docs.docker.com/compose/compose-file/#ports)
    - If using ember, wait until the server is completely done building before
        requesting.
    - Make sure the service in the docker container is not bound to 127.0.0.1
    - You need to run `lsof` with `sudo` on your docker machine.
    - Test the connection using `telnet <host> <port>`, from inside the docker
        container, from docker machine, and from your osx host.
    - Running `docker ps` will show you which ports are being forwarded by
        docker to the docker-machine.
* `lxc-checkconfig` checks for missing kernel features requested by lxc.
* Most `lxc-*` commands take `--logfile=lxc.log` and `--logpriority=TRACE` as
    options, in case you can't figure out what's going on from the obscure
    error message you get, or everything looks dandy but it didn't do what you
    want.
* If you want to run your CI system in docker to build docker images,
    bind-mount the host docker daemon socket into the CI docker container. Your
    life will be better for it. [See this article](http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
* Docker-machine process:

    ```
    machine_name=default
    docker-machine start $machine_name
    eval $(docker-machine env $machine_name)
    ```

# system

* `otool -L` is the OSX equivalent of `ldd`
* `ldd` - ldd prints the shared objects (shared libraries) required by each
    program or shared object specified on the command line.
* To view system variables, run `sudo sysctl -a`.
* If a program is being called when you don't expect it, here's a dirty way of
    finding out what is happening:

    ```
    #!/bin/sh
    ps axf > /tmp/apt-get.$$
    apt-get.real "$@"
    ```
* To view stats on last ntp sync, use `ntpq -p`. See
    [this page](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s1-Checking_the_Status_of_NTP.html)
    for details.
* to view configuration for systemd unit,

    ```
    systemctl cat <unit>
    ```
* OOM killer will favour killing shorter-lived processes than long ones, to the
    tune of

    ```
    process_badness =                      mem_used_by_process
                      ----------------------------------------------------------------
                      sqrt(process_lifetime_in_s))*sqrt(sqrt(process_lifetime_in_min))
    ```

    See [chapter in kernel book](https://www.kernel.org/doc/gorman/html/understand/understand016.html)
* OOM killer checks for available memory in page cache, free pages, free swap
    pages, and a couple other caches before deciding there is no more memory
    available.
* `swapon -s` to test if you have swap enabled
* Brendan Gregg lists some [useful commands](http://www.brendangregg.com/USEmethod/use-linux.html)
    to apply USE to various aspects of unix resources such as CPU, memory,
    network, etc.
* There's no easy way of asking to see all the hard links to a file. `find /
    -inum <inode>` is a pretty good way of seeing them.
* Directories always have at least 2 hard links that reference them, its full
    path and `.`. It gets an additional hard link for every child directory,
    because of the `..` hard links.

    ```
    $ mkdir foo
    $ ls -ld foo
    drwx------  2 marguerite  staff  68 Nov 24 10:18 foo/
    $ mkdir foo/bar && ls -ld foo
    drwx------  3 marguerite  staff  102 Nov 24 10:18 foo/
    $ mkdir foo/baz
    $ ls -ld foo
    drwx------  4 marguerite  staff  136 Nov 24 10:19 foo/
    ```
* The 2nd column of `ls -l` shows the number of hard links to the file.
* To see the *actual* groups of a running process (not the groups that the user
    running the process *should* have), use:

    ```
    grep Groups /proc/$$/status
    ```
* `pv`, "Pipe Viewer", is basically `cat`, but it tracks how much data has come
    through, and when possible, how much is left. The following examples are
    [from this nice blog post about pv](http://www.catonmat.net/blog/unix-utilities-pipe-viewer/):

    ```
    pv access.log | gzip > access.log.gz
    pv -cN source access.log | gzip | pv -cN gzip > access.log.gz
    tar -cf - . | pv -s $(du -sb . | awk '{print $1}') | gzip > out.tgz
    ```

    You can use it to inspect the speed of the network as well:

    ```
    # on computer A, with IP address 192.168.1.100
    $ tar -cf - /path/to/dir | pv | nc -l -p 6666 -q 5
    ```

    ```
    # on computer B
    $ nc 192.168.1.100 6666 | pv | tar -xf -
    ```
    (Note this is backwards from how I usually do it -- unsually I would have
    computer A initiate the connection, and have computer B listen for traffic.
    But it all works out in the end.)
* USDT is "user-level statically-defined tracing". It is where a probe has been
    inserted & compiled into code. Tools like dtrace can then hook into those
    probes.
* A uprobe is a probe inserted into user-space code
* A kernel probe is a set of handlers placed on a certain instruction address.
    * A KProbe is defined by a pre-handler and a post-handler.
    * JProbes are used to get access to a kernel function's arguments at
        runtime. A JProbe is defined by a JProbe handler with the same prototype
        as that of the function whose arguments are to be accessed.
* Steps to proper daemonization of a process:

    1. fork
    1. setsid
    1. fork
    1. chdir("/")
    1. umask(0)
    1. close fd 0, 1, 2, all other file descriptors
    1. reopen 0, 1, 2

    See [this list](http://web.archive.org/web/20120914180018/http://www.steve.org.uk/Reference/Unix/faq_2.html#SEC16)
    and [this more in-depth explanation](http://codingfreak.blogspot.com/2012/03/daemon-izing-process-in-linux.html)
* If you `rm -rf` a filesystem mount, you get `EBUSY`
* /var/run in ubuntu 14.04 is actually /run, which is tmpfs. This means it gets
    blown away each boot. So if your service writes a pid file to
    `/var/run`, you should probably `mkdir -p /var/run/needed-dir` and `chown
    <service_user> /var/run/needed-dir`. Otherwise your service won't be able
    to write its pid.
* If you write an init script that hangs, and it runs on boot, boot will hang.
    If it runs on shutdown, shutdown will hang. Make sure you test your init
    script in odd scenarios so that this doesn't happen to you.
* size footprint of all top-level directories, sorted by size:

    ```
    du -sh /* | sort -hk1
    ```
* You can't very well strace `service xyz restart` if it's controlled by
    upstart. Upstart uses events to trigger the running of applications, so your
    application won't be a fork / exceve of `service xyz restart`.
* The below two points make the process model for (say) `unicorn` work. The
    parent root `unicorn` process is typically run as root, whereas the children
    are often not. But the children can still access the log files created by
    the parent.
* When a process forks, the file descriptors are inherited by the children, and
    they point to the same file description in the OS's file table.
* If you open a file and then `seteuid` so that the new effective user is not
    the owner of the file, the process can still operate on the file descriptor.
    via the original executing user's privileges.
* The shebang line in a program tells the program executor to run the program
    specified in the shebang line, passing the file as an argument. E.G.:

    ```
    $ ./meow
    #!/bin/cat
    meow
    ```
* When debugging a command with `strace`, first make sure the behaviour is the
    same between when you `sudo` or you don't `sudo` the command.
* `man proc` shows you all the things you can find under `/proc`
* Bind mounting lets you mount the same filesystem at 2 different spots
* Show total amount of free memory: `free -m` (`-m` is for "megabytes")
* `-y` option for `strace`: displays full path in file descriptors. e.g.:

    ```
    [pid 28968] read(244</var/go/x.properties>, "prop1=stuff\nprop2=stompy"..., 8192) = 72
    ```
* [`sysdig`](http://www.sysdig.org/) is strace+tcpdump+lsof in one more
  user-friendly package.
* UID mappings. A single user can own a range of userids, useful for user
  namespaces. In `/etc/subuid` (see `man subuid`), you   can see what range
  each user has access to.
* the environment variable `LD_LIBRARY_PATH` defines the search path for
    dynamically linked libraries

# command line

* If you want a command to read from stdin, but it only accepts file arguments,
    specify `/dev/stdin` as the file. Example:

    ```
    echo foo | ssh-keygen -lf /dev/stdin
    ```

    This might not work because `/dev/stdin` is not a regular file in this case.
    However if you use Bash's heredoc, it should work.
* `xargs -J % <command containing %>` will replace the first occurence of `%`
    with the argument coming from stdin, instead of appending the argument to
    the end of the command. This is useful when the argument of the command you
    are running doesn't go at the end.
* You can get `curl` to make requests following a set alphanumeric sequence by
    adding something like `[1-5]` in the URL. eg `curl localhost/[1-5]`. Also
    works with set expansion: `curl localhost/{foo,bar,baz}`. See the URL
    section (near the top) of [the curl man
    page](https://curl.haxx.se/docs/manpage.html).
* `watch -t '(date; <some-command>)' | tee -a <file>` will log the date and
    output of running a command every second. The following is maybe easier to
    remember:
    ```
    while true; do <some-command>; sleep 1; done | while read line; do echo $(date) $line; done
    ```
* `shuf` (or `gshuf` on a mac when you `brew install coreutils`) can be used
    for extracting a random line from a file.
* `lastlog` shows the last login date for each user.
* `last` shows the history of who has logged into a box.
* `echo "foo.bar.baz:1|c" | nc -w 1 -u <statsd_bridge> 8125` to manually emit
    statsd metrics
* Converting a unix timestamp: `date -d @<unix-timestamp>`
* On ubuntu, `sort -h` sorts by "human readable" sizes, e.g. `1G` vs `1K`.
* `pbpaste` is an OSX command to paste from your buffer (goes along with
    `pbcopy`).
* On ubuntu, `envsubst` allows you to expand environment variables in text from
    standard input. You can specify which environment variables to expand using
    the argument to `envsubst`.

    ```
    $ FOO=1 BAR=2 BAZ=3 envsubst '$FOO,$BAZ' <<< '$FOO $BAR $BAZ'
    1 $BAR 3
    ```
* To make spaces into newlines on command line: `tr ' ' "\n"`
* One way to parallelize processes on the command-line is to use xargs:

    ```
    find . -name '*.gem' | xargs --max-procs=5 --max-args=1 gem install
    ```
* A cheap way to get an audible bell that will alert when the current command
    finishes running: enter a tab twice! when the command is finished, it will
    try to tab complete, causing a bell.
* `tail +2` will print all but the first line of the file.
* Some cheap ways to truncate a file:
    * `truncate -s 0 $dest_file`
    * `cp /dev/null $dest_file`
* OSX puts its text dictionary in `/usr/share/dict/words`
* You should stop using `ps aux`. See [stackexchange
    question](http://unix.stackexchange.com/questions/106847/what-does-aux-mean-in-ps-aux).
    OSX special-cases the `aux` argement, which means you
    can't easily add another option like `-f` in the `aux` form. The preferred
    form, `ps -ax`, allows you to add the `-f` easily.

# bash

* To exit with an error message if a variable is not set or is empty:

    ```
    ${foo:? error}
    exits with status 1, with this output: bash: foo:  error
    ```
* Parameter substitution documentation [lives here](http://www.tldp.org/LDP/abs/html/parameter-substitution.html)
* `#!/bin/bash -x -e` is not a valid shebang line on ubuntu (it is on osx).
    The valid shebang line is `#!/bin/bash -xe`
* `set -u` in bash causes an error when you use an undefined variable.
* In bash, `seq 1 1 15` will print numbers 1 to 15.
* To manage a bash command with an infinite loop:

    ```
    echo $$ > /var/run/my_script.pid
    ```
    and then, somewhere else, `kill $(cat /var/run/my_script.pid)`
* `1>&2` at the end of your line to direct stdout to stderr. STOP FORGETTING!
* In bash, to use `<` to compare strings, you should use `[[`, and not `[`:

    ```
    $ [ a > b ]   # exits 0
    $ [[ a > b ]] # exists 1
    ```
* In bash, if `~` is (double or single) quoted, it doesn't get expanded to your
  home directory.

    ```
    $ file ~
    /Users/marguerite: directory
    $ file "~"
    ~: cannot open `~' (No such file or directory)
    ```

# git, github

* Use [the github search documentation
    page](https://help.github.com/articles/searching-code/) to learn all the
    fancy things you can do with search. The [advanced search
    page](https://github.com/search/advanced) doesn't have nearly as many useful
    keywords.
* `git checkout -` works like `cd -`, checking out the previous thing you had
    checked out.
* `git diff <branch>:<file> <branch>:<file>` lets you diff files across
    branches
* `git checkout <tree-ish> -- path/to/file` checks out and adds to index the
    version of `path/to/file` as it was in `<tree-ish>`.
* `git rebase --exec <cmd>` runs `<cmd>` for every commit that you rebase.
    (First seen [here](https://disjoint.ca/til/2017/10/04/signing-your-git-commits-using-your-gpg-key/))
* `git commit --amend --no-edit` says to use the commit message for the commit
    you're about to amend, instead of forcing you to edit it.
* Add this to your `.git/config` if you want `git fetch` to pull down all the
    pull request tips. Put it under `[remote "origin"]`.

    ```
    fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
    ```
* Gists, though very git-repo-like, don't support directories:

    ```
    $ g push origin master
    Counting objects: 47, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (47/47), done.
    Writing objects: 100% (47/47), 1.31 MiB | 113.00 KiB/s, done.
    Total 47 (delta 7), reused 0 (delta 0)
    remote: Gist does not support directories.
    remote: These are the directories that are causing problems:
    remote: slides
    To git@gist.github.com:<name>.git
     ! [remote rejected] master -> master (pre-receive hook declined)
    error: failed to push some refs to 'git@gist.github.com:<name>.git'
    ```
* The `GIT_SSH` environment variable is how you override the command that git
    is using to ssh. Set `GIT_SSH` to the path of a file that specifies how the
    ssh command should be invoked. An example of such a file:

    ```
    ssh -i /home/stompy/.ssh/id_rsa "$@"
    ```
* To move a patch between git repositories:
    creating the patchfile:

    ```
    git diff -p > x.patch
    ```

    Applying:
    ```
    git apply x.patch
    # or
    `patch -p1 < x.patch`
    ```
* If your automation experiences a timeout during a `git clone` on `git
  ls-remote`, it's possible that `git` is expecting user input, such as a
  password to decrypt your private key (which in my case was empty!)
* In GitHub, if you are looking at "foo.rb on master", you can hit "y" to get
    you a link to "foo.rb at that specific commit". Helpful if you want to link
    to a specific line, you can do so without worrying about the lines shifting
    over time as master changes

# chef

* To search for a chef node that does _not_ have an attribute:

    ```
    knife search node "NOT my_attribute_name:*"
    ```
* To do a more complex command via knife ssh, put the whole command in quotes:

    ```
    knife ssh 'name:foo*' "sudo bash -c 'FOO=BAR echo hi'"
    ```
* In chefspec, you can parse contents of a template to test the structure like so:

    ```
     expect(subject).to create_template('foo').with_content do |content|
       j = JSON.parse(content)
       expect(j).to_not be_nil
       expect(j['bar']).to eq('baz')
     end
    ```
* You can set up an irb environment to be very similar to the environment in
    which Chef executes:

    ```
    require 'chef'
    Chef::Config.from_file '/path/to/knife.rb'

    Chef::Node.list

    node = Chef::Node.load('stg-ci-agent24') ; nil
    node.name
    node.recipes

    q = Chef::Search::Query.new
    ci_agents = q.search(:node, 'name:stg-ci-agent*') ; nil
    ci_agents.size
    ci_agents.first
    ci_agents[0].first.name
    ci_agents[0].first.kernel
    ```
* To see hosts' pretty hostnames when you `knife ssh`, use `knife ssh -a fqdn`.
* To run a single chef recipe on a node: `chef-client -E <chef_env>  -o "recipe[<recipe_name>]"`
* To indicate attribute nesting to the _search string_ of a `knife search`, use
    underscores. But to indicate attribute nesting to the `-a` flage, use dots.
    ```
    knife search node 'foo_bar_baz:true' -a foo.bar.baz
    ```
* You can read the parsing for chef / knife search strings in [this chef zero
    file](https://github.com/chef/chef-zero/blob/a9f0654dabb6aa0f3c59e66abc1a36bfd7767947/lib/chef_zero/solr/solr_parser.rb)
* knife searches:

    ```
    knife search node '( recipes:foo OR recipes:foo\:\:default ) AND roles:bar'
    ```

    You have to escape `:` because it's a special character.
* Chef backs up files before it changes them to `/var/chef/backup`
* In chef specs, if atttributes which have defaults are appearing as nil, it might
    be because the attributes belong to a dependent cookbook which is not being
    declared as a dependency in `metadata.rb`.
* If you get this error from chef and the client & node aren't in `knife
    {client,node} list`, you may need to delete `/etc/chef/client.pem` on the
    node.

    ```
    Failed to authenticate as 'pd-ci-agent1'. Ensure that your node_name and client key are correct.
    ```
* When you create an LWRP in chef, you can create a chef-spec "matcher" for it.
    For example, you would be able to write this in a spec file:

    ```ruby
    expect(chef_run).to my_custom_matcher('...')
    ```

    The name `my_custom_matcher` is chosen by you, and specified in
    `libraries/matcher.rb` in your coobkook. See [more
    info](https://github.com/sethvargo/chefspec#packaging-custom-matchers)
* In chef, if you get an error like this, you may have overridden your
    `/etc/hosts` during the chef run so that your hostname does not resolve to
    what it should anymore:

    ```
    Server Response:
    ----------------
    Failed to authenticate as 'stuff'. Ensure that your node_name and client key are correct.

    Relevant Config Settings:
    -------------------------
    chef_server_url   "http://10.0.2.15:4646"
    node_name         "stuff"
    client_key        "/etc/chef/client.pem"

    If these settings are correct, your client_key may be invalid, or
    you may have a chef user with the same client name as this node.
    ```
* [This](https://github.com/nickola/chef-hostname/blob/595016a90bc66e548410f8882f59e15eeee6f180/recipes/default.rb#L49-L65)
    is a good example of how to do search-replace-or-append-line in chef.
    Maybe you should make that into an lwrp :P
* Attribute nesting in chef:
    > `ohai foo/bar`
    > `search foo_bar`
    > `knife foo.bar`
    > these are the three forms for specifying nesting..
* The chef `search` function uses `solr` as a backend. So you can't use `.` to
    describe nested attributes, because the `.` is already used for something
    in solr
* By default, `berks` (the `chef` equivalent of `bundler`) looks to
    http://cookbooks.opscode.com/ to download cookbooks listed in `Berskfile`.
* In ChefSpec, notifications don't fire, since ChefSpec is attempting to stub
    out any actions that are taken as a result of a resource. This means you
    can't test something with `action :nothing`, since it won't get triggered.
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
* An LWRP in chef is a Lightweight Resource or Provider. Chef provides 2 ways
  of implementing a resource (say). You can subclass a Chef::resource and write
  your own getters and setters, or you can define a resource using a simple DSL.
  [See this page](https://docs.chef.io/lwrp_custom.html)
* [goiardi](https://github.com/ctdk/goiardi) is an in-memory chef server
  written in go
* To search for which `chef` nodes have a certain role (e.g. `foo`), use `knife
  search node role:foo`
* `knife` is a tool to change files in a chef-server at runtime. It can be run
  from a machine that isn't the chef server - it communicates updates to the
  server using an HTTP API. E.g.:

    ```bash
    knife node edit my-node01  # lets you edit the configuration for this node
    ```
* To run chef on a client machine, `sudo env -i chef-client`

# golang

* To list dependencies in a Golang project: `go list -f '{{ .Deps }}'`
* in go,

    ```
    x = some_go_struct{}
    *x.some_string_pointer = "stompy"
    ```
    is not allowed - null pointer!

    ```
    var s string
    x = some_go_struct{
      some_string_pointer = & s
    }

    *x.some_string_pointer = "stompy"
    ```
    is allowed
* Testing in go: Say you want to write a test for the package "foo". In the
    directory where you have the foo package, create nother file `foo_test.go`.
    Fill it with this:

    ```
    package foo

    import "testing"

    func TestBar(t *testing.T) {
      v = Bar([]float64{1,2})  // bar is defined in `foo`
      if v != "stompy" {
        t.Error("Expected 'stompy', got ", v)
      }
    }
    ```
* Running `godoc -http=":6060"` makes browsable all the docs on your system,
    at [localhost:6060/pkg/](http://localhost:6060/pkg/)
* In golang, functions that begin with a capital are visible to programs that
    import the packaged.
* You can make a channel with a limited buffer size. Goroutines will block if
    the buffer is full.

    ```go
    c := make(chan string, 1) // buffer size of 1
    ```
* Goroutines communicate over a "channel", which appears to be a pipe with a
    type.

    ```go
    func pinger(c chan<- string) {
      c <- "ping"
    }

    func printer(c <-chan string) {
      fmt.Println(<-c)
    }

    func main() {
      c chan string = make(chan string)
      go pinger(c)
      go printer(c)
    }
    ```

    Omit the arrows to make the channel bidirectional.
* A `goroutine` is created when you make a function call but prefix it with
    `go`.

    ```
    go foo()
    go bar()
    ```

  These functions will execute concurrently.
* Interfaces in golang:

    ```
    type Shape interface {
      area() float64
    }
    ```
    You can use interfaces as types.
    Any types that already have methods corresponding to that interface
    automatically implement that interface.
* Types in go implement the is-a relationship as so:

     ```go
     type Foo struct { x float64 }
     func (f *Foo) zip() { /* something */ }

     type Bar {
       Foo
       y float64
     }

     b := new(Bar)
     b.Foo.zip()
     b.zip()
     ```
* Methods on structs in go:

    ```go
    func (c *Circle) area() float64 {  // "c" is the "receiver"
      return math.Pi * c.r*c.r
    }
    ```
* Structs / types in go:

    ```go
    type Circle struct {
      x float64
      y float64
      r float64
    }
    ```
* Functions seem to be first-class in go. You can return them, create inline
    functions with closure...
* Use `defer` in `golang` to "ensure" something gets called before the function
    returns.
* You can name the return value in a golang `func`:

    ```go
    func x() (my_return int) {
      my_return = 5
      return
    }
    ```
* Function declaration in golang:

    ```go
    func <name>(<params>) <return type> { }
    ```
* iterating over an array in golang:

    ```go
    for _, v := range my_slice {
      stuff
    }
    ```
* When you access a map key that doesn't exist, go returns the "zero" value of
    that type. Or you can check for the existence like this:

    ```go
    if name, ok := my_map["doesn't exist"]; ok {
      fmt.Println(name, ok)
    }
    ```
* Adding a nd deleting from a golang map:

    ```go
    x["hi"] = 1
    delete(x, "hi")
    ```
* Declaring a `map` in `golang`:

    ```go
    var x map[string]int   // for some reason x needs to be initialized
    x := make(map[string]int)
    ```
* To declare a `slice` (the more useful version of an `array`) in go:

    ```go
    var x []int  // length 0
    x := make([]int, 5)  // slice of length 5
    ```

    Also useful are `copy` and `append`.
* `golang` variable declaration: `var <var-name> <type> [= <initialization> ]`

    ```go
    var x string = "hello"
    ```
    You can also do it this way, and `go` infers all the ommitted stuff:

    ```go
    x := "hello"
    ```
* `godoc <package> <function>` prints the docs for that function.
* The file extnsion used for Golang templates in the docs is `.tmpl`.
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
* [Summary](https://www.golang-book.com/books/intro/13) of useful golang
    library functions

# bundler, ruby, rake, rspec, rvm

* the environment variables `BUNDLE_PATH`, `BUNDLE_APP_CONFIG` and `BUNDLE_GEMFILE` can be set for non-standard bundle file hierarchies
* `rvm cleanup all` removes all stale sources and archives.
* bundler's `cache_path` config option, or the `BUNDLE_CACHE_PATH` environment
    variable, tells bundler where to put (and also where to look for) the bundle
    cache generated with `bundle package` and `bundle cache`
* The ruby equivalent of python's `repr()` is `.inspect`. `puts "".inspect`
    will print `""`, instead of nothing.
* You can tell bundler to pass certain flags during the installation of certain
    gems. This configuration exists per-machine. For example,

    ```
    bundle config build.mysql --with-mysql-config=/usr/local/mysql/bin/mysql_config
    ```
    This will cause the following invocation of `gem` during a `bundle install`:

    ```
    gem install mysql -- --with-mysql-config=/usr/local/mysql/bin/mysql_config
    ```
* [This page](https://github.com/bundler/bundler/blob/master/ISSUES.md) is
    helpful in giving hints to solving bundler problems:
* If you get errors like these:

    ```
    pd_lite@vagrant-ubuntu-trusty-64:~/numerator$ /opt/ruby/2.2.2/bin/bundle exec cap pdlite report:check_current_version
    Could not find highline-1.7.8 in any of the sources
    Run `bundle install` to install missing gems.
    ```
    after you have run bundle install, watch out that the bundler version you
    are using is compatible with other things (like gem and ruby) in your path.
    For example, I got that error because I was invoking a bundler that was not
    on my path, which belonged to a different ruby than the one that was on my
    path. Putting `/opt/ruby/2.2.2/bin` first in my path fixed that particular
    error.
* `bundle package --no-install` downloads all gems to a local cache without
    installing them. This is useful if you want to download all gems as a
    pre-build step for building a docker image. (useful if some gems come from
    private git repos.)
* To assert the conjunction ("and"-ing) of two rspec matchers, use the `.and`
    method of a matcher:

    ```ruby
    "abc".should(include("a").and(include("b")))
    expect(foo).to receive(:bar).with(include("slurp").and include("burp"))
    ```
* `rake <task> --dry-run` will show you all the tasks that will be executed to
    run <task>.
* To create a negated rspec matcher, do the following:

    ```ruby
    RSpec::Matchers.define_negated_matcher :not_match, :match
    expect(x).to not_match(/stompy/)
    ```
* A [matcher](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers)
    in rspec is the part that comes after the `expect(x).to `. Eg,

    ```ruby
    expect(x).to match(/slurp/)
    ```
* To run a specific test using the `Test::Unit` `ruby` framework, you can do
    this:

    ```
    bundle exec ruby -I test path/to/test_file.rb -n '/method_name_regex/'
    ```
    In this case, `/method_name_regex/` should match `it_does_what_you_want`,
    assuming you have a test that looks like this: `it 'does what you want'`
* `bundle install --retry=5` retries 5 times on network and github issues
* `bundle install --jobs=5` starts 5 parellel workers
* Brightbox has sensible ruby packages for ubuntu.
    [See the docs about it](https://www.brightbox.com/docs/ruby/ubuntu/)
* When you run `gem source --add <url>`, your `~/.gemrc` gets updated.
    That means if you run that as root, as (say) part of a chef run, non-root
    users will not be able to see that added source.
* When you get the following type of error:

    ```
    $ bundle exec irb
    Could not find rake-10.4.2 in any of the sources
    Run `bundle install` to install missing gems.
    ```
    be sure that bundler is loading the correct irb. It could be that `rake` is
    installed under `ruby2.0.0`, but it is trying to run an `irb` for an older
    version of ruby. Try
    ```
    $ bundle exec irb2.0
    ```
    Not completely clear on why this happens, but it will likely go away if you
    unlink /usr/bin/irb and relink it to /usr/bin/irb2.0.
* Before you package a gem, ensure the permissions of the files that will go
    into your gem are world-readable. Pre-packaged permissions carry through to
    post-installation. If someone runs `sudo gem install <yourgem>` to
    install your gem system wide, if the files originally lacked
    world-readability, non-root users will not be able to use your gem.
* If you mount a bundler project into a VM, make running bundler faster by
    installing the bundler dependencies in another directory on the VM.
* In `rspec`, to get a non-strict double (one doesn't care what method you
    call) that returns a non-strict double for any method you call on it, use

   ```ruby
   double().as_null_object
   ```
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
        to build your gem, and `rake release` to publish your gem on rubygems

        ```
        Bundler::GemHelper.install_tasks
        ```
* In ruby documentation, `Klass.method` is referring to the class method
    `method` on `Klass`, and `Klass#method` is referring to the install method
    `method` on `Klass`.
* Multi-line comments in ruby:

    ```ruby
    =begin
    ^ should be at the beginning of the line
    This entire thing is a comment!
    =end
    ```
* How to break up long strings in ruby:

    ```ruby
    x = 'this '\
      'is a kinda '\
      'long string'
    ```
* The environment variable `BUNDLER_GEMFILE` tells bundler which Gemfile to
    use, so you don't need to `cd` somewhere to run `bundle exec`.
* In ruby, you can do a set-like difference of arrays like this:

    ```ruby
    x = [1, 2, 3]
    y = [3, 4, 5]
    x - y
    ==> [1, 2]
    ```
    There is also the Set class, if you're into that sort of thing.
* The Net::HTTP lib lets you make web requests in ruby
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
* To test a given file in `rspec`:

    ```
    rake spec SPEC=/path/to/spec_file.rb
    ```
* In `rspec` tests, `context` is the same as `describe`, and `describe` is a
    thing that lets you apply `before` and `let` blocks to a group of tests.
    You can nest `describes`, but it's easier to read if you use `context`
    inside of a `describe`.
* Why `let` is better than instance variables in `before` blocks in `rspec`
    tests (See [this stack overflow answer](http://stackoverflow.com/questions/5359558/when-to-use-rspec-let/5359979#53599790)
* One way of creating ruby libraris: In your library base file you should
  require all the files defining things you want at top-level scope. E.g.:

    ```ruby
    # lib/my-library.rb
    require 'my-library/version'

    Dir[File.join(File.dirname(__FILE__), 'my-library', 'top-level', '*.rb')].each { |lib| require lib }
    ```
* You need to install the `ruby-build` plugin for `rbenv` if you want to be
  able to `rbenv install 2.1.4`
* To see the actual executable you are using while under rbenv shims magic, use
  `rbenv which gem`

# python

* [Manipulating binary data in python](http://www.devdungeon.com/content/working-binary-data-python)
* `list('abc')` => `['a', 'b', 'c']` in python (list of a string makes a list
    of chars)
* `set('abc')` => `set(['a', 'b', 'c'])` in python (set of a string makes a
    list of chars)
* `enumerate(alist)` will give you `index, item` when iterating over it.


# aws

* Go to https://phd.aws.amazon.com to see all your retirement notices at once.
    (Personal health dashboard)
* [list of names of datacenters and where they are](http://docs.aws.amazon.com/general/latest/gr/rande.html)
* On an AWS node, `curl http://169.254.169.254/latest/meta-data/` gives you all
    its instance metadata / attributes! My coworker says:

    > The instance metadata service is great for self-provisioning instances.
    > If we were to ever use auto-scaling, you can just have a simple startup
    > script which reads in things like the instance tags from the metadata
    > service, and provisions itself based on the tags it has.
    >
    > e.g. we have a single base AMI, but based on how we tag the instance
    > would determine what it provisions as. So you can have an auto-scaling
    > group set tags like `app=load-balancer`, `env=prod`, etc. and it can
    > provision itself accordingly.
    >
    > It has some security issues though. Since the instance metadata service
    > also returns API tokens if you use EC2 roles, anything on the box with
    > network access can query the service and get those temporary tokens.
* Testing who the aws credentials belong to:

    ```
    pip install awscli
    AWS_ACCESS_KEY_ID=foo AWS_SECRET_ACCESS_KEY=bar aws iam get-user
    ```
* AWS lambda debugging: If a lambda function test is giving you unhelpful
    errors,
    * Check that all files in the uploaded zip are world-readable
    * Check that all directories in the uploaded zip are world-executable.
* When you get weird error messages from AWS saying "invalid value for field
    groupId" as part of a RunInstances action, I suspect it's because the
    security group name you provided does not exist. Check that you are using
    the correct aws credentials!
* `--include` and `--exclude` for `s3cmd sync` does not work the way I expect.
    Would have thought that `--exclude '*' --include '*/foo/console.log'` would
    pick up all `foo/console.log`. But in actuality I first need to `include`
    whatever directories `foo` might happen to be in.  At least that was my
    observation today.

# mysql

* For MySQL configuration / plist on OSX, [this](https://gist.github.com/kevinelliott/e12aa642a8388baf2499#mysql-settings)
     might be helpful
* User management commands in mysql:

    ```
    DROP USER 'username'@'%'
    CREATE USER 'username'@'%' IDENTIFIED BY 'mypass';
    GRANT ALL ON *.* TO 'useranem'@'%';
    REVOKE INSERT ON *.* FROM 'username'@'%';
    SET PASSWORD FOR 'username'@'%' = PASSWORD('mypass')
    ```
* in mysql, if you have a `''@'localhost'` user, a `'marguerite'@'%'` user and
    NO `'marguerite'@'localhost'` user, then the `''@'localhost'` will take
    precedence. You might get permission denied if you try to use mysql on
    localhost using the `marguerite` user with `marguerite`'s password. If
    neither `''` or `'marguerite'` has a password, you might get `''`'s grants.
    Basically, you need to greate a `'marguerite'@'localhost'` user with
    `'marguerite'`'s password for `marguerite` to be able to log in on localhost
* Use `--defaults-file=` (`=` is required) for specifying a mysql config file

# logging

* `journalctl /path/to/executable` streams logs from that executable
* `journalctl --unit <service>` is how you view the logs for a service that
    uses systemd.
* Upstart scripts log service execution to `/var/log/upstart/service-name.log`.
* `upstart` logs to `/var/log/syslog` when respawning a process:

    ```
    Feb  9 21:30:30 <host> kernel: ] init: foo main process (<pid>) terminated with status 1
    ```
* Testing `logrotate`:
    - `logrotate -d` gives good debug output wrt what `logrotate` is doing.
    - `logrotate --fonce` forces a logrotate run even if it's not needed.
    - `/var/lib/logrotate/status` is a log when rotates happened. You can edit
        it to trick logrotate into rotating again.

# vim

* When you yank text in vim, the latest yank is stored in the `0` register.
    The 2nd to last yanked text in `1` register.
* To paste from a register, use `"<register-character>p`. To yank into a
    register, use `"<register-character>y{motion}`. Also works whene deleting
    with `d`.
* `ip` is a motion in vim that means "in paragraph". `yip` yanks the paragraph,
    `dip` deletes a paragraph, etc. Works like `i(`, `i"`, etc.
* Never again worry about deleting a file whose name is only punctuation,
    possibly magic punctuation that might be interpreted by your shell. Delete
    the file in vim's explore mode:
    * `vim .`
    * Move cursor over the file you want to delete
    * hit 'D'
* In `vim` "explore" mode, hitting `t` will open the file under the cursor in a
    new tab.
* In vim, `:ls` lists your buffers. `:b<n>` takes you to buffer `<n>`.

# tmux

* Tmux shared sessions:
    * *you*: `tmux -S /tmp/stompy.tmux.sock new stompy`
    * *them*: `tmux -S /tmp/stompy.tmux.sock attach -t stompy
    * They will either have to sudo, or you will have to chmod / chown the
        `/tmp/stompy.sock` to be readable by them.
* If your tmux is screwed up because you accidentally catted a binary file,
    you may be able to fix it by renaming the window in which you catted.
    [See the stack exchange question.](http://unix.stackexchange.com/questions/49886/tmux-status-bar-corrupted-after-catting-a-binary-file-how-to-reset)
* In tmux to clear your scrollback history: `clear-history`
* In tmux, if you `C-B` and immediately hit the up arrow key multiple times,
    you will cycle through the window panes.

# Frontend development

* [codepen.io](https://codepen.io) lets you share an HTML/CSS/JS application.

# Google Sheets

* Relative cell references in custom formulas for conditional formatting:
  specify the formula relative to the top left cell of the conditional
  formatting range.
* Conditional formatting custom formulas: You can use the standard formulas
  that you would use in a cell. Start with an `=`.
* In a formula, if you DON'T use `$` anywhere, the cell reference you use is
  RELATIVE in terms of offset rows and columns from the current cell. So if you
  drag the corner of the cell to fill in other cells, the row & column reference
  will change in those other cells to have the same relative offset WRT _those
  cells_. If you want an _absolute reference_ for (eg) a column (which is
  identified with letters), so that the column reference keeps the same letter
  even if you drag the corner of the cell to fill other cells, you must use a
  `$` in front of the column reference. Eg $A1 will fix the column reference.
* Google Sheets has a "paste column width only" option.

# everything else

* In slack, to apply formatting to a message that already has formatting
  characters present, type Command+Shift+F.
* In Google Calendar, there is a more straightforward way to show/hide declined
  events than going into Settings. Near top right, there is a dropdown for what
  timerange to display. Click that, and at the bottom, there is "show/hide
  declined events".
* If you shorten a URL with [Google's URL shortener](http://goo.gl/), you can
  track how many times the link was clicked. [help
  doc](https://support.google.com/faqs/answer/190768?hl=en)
* To extract text from an image, upload it to google drive, then right click >
  Open with google docs. Text will be automatically extracted.
* Chrome Apps - a way to have a webpage open as its own application, without all
  the distraction of tabs and address bar up top, and you can add the shortcut
  to your doc. To create a shortcut: Open the webpage, click the three circles
  in the top right of your browser > More tools > Create shortcut > Check "open
  as window". I can work on my google doc with less distraction now!
* Fixing my nicks on IRC when using irssi & bouncer:
    https://disjoint.ca/til/2016/04/25/useful-znc-commands/
* The Nomad UI has a [demo site](https://demo.nomadproject.io/ui/jobs)
* Cmd+ctrl+Q to lock your screen on OSX
* downforeveryoneorjustme.com/<another-url> is a way of checking if another
    party sees something as being down
* [Here are the available functions to apply to your datadog
  metrics](https://docs.datadoghq.com/graphing/miscellaneous/functions/)
* [Here is the API Specification for creating a datadog
    monitor](https://docs.datadoghq.com/api/?lang=python#create-a-monitor)
* [Here is the order](https://help.datadoghq.com/hc/en-us/articles/204820019-Graphing-with-Datadog-from-the-query-to-the-graph)
    in which datadog evaluates the different parts of a query when generating a
    graph.
* Datadog's [`rollup` function](https://help.datadoghq.com/hc/en-us/articles/204526615-What-is-the-rollup-function-)
    affects how things are graphed. I'm not sure why you would need it in a
    monitor.
* In Jira's JQL, `(+/-)nn(y|M|w|d|h|m)` is how you specify a time relative to
    now, or a time increment relative to the date function it's being passed in
    to.

    ```
    created >= "-5d"
    endOfMonth(-6M)
    ```
* In `sqlite3`, `cast(substr(location, 1, instr(location, "-") - 1) as integer)`
    will split a string on `-`, take the first part, and cast to int.
* In `sqlite3`, `.schema <table>` shows you the table schema.
* In `sqlite3`, `.tables` shows you the tables in the db.
* `$('.class-name')` is the jquery selector for things of a certain class
* `$('ancestor-selector descendent-selector')` is the jquery selector for
    something that's a descendent of something else
* To view the health of a cassandra node, try `nodetool status` and
    `nodetool netstats`.
* To authenticate to the slack api, you have to pass the "token" parameter
    either as a query string in the URL or as part of the POSTed JSON:

    ```
    https://slack.com/api/search.messages?token=<your-token-here>
    ```
    or

    ```
    {
      "token": "your-token-here",
      ...
    }
    ```
* You can do complex searching on JIRA issues using JQL, Jira query language.
    Jira > project page > all issues and filters > advanced. Example:

    ```
    project = STOMPY AND status not in (open) AND resolved >= startOfYear() AND resolved <= endOfYear() ORDER BY updated ASC
    ```
* USE is a method for understanding a performance problem. It stands for
    Utilization Saturation Errors.
* concatenating two gzips makes a single valid gzip!
* TextEdit saves recovery files here: `~/Library/Containers/com.apple.TextEdit/Data/Library/Autosave\ Information/`
* Sending an event to datadog:

    ```
    title=foo
    text=bar
    echo "_e{${#title},${#text}}:$title|$text|#shell,bash"  >/dev/udp/localhost/8125
    ```
* Invoking a serf handler from the command line:

   ```
   echo <payload> | env SERF_QUERY_NAME=bar handler.sh
   ```
   See [the docs](https://www.serfdom.io/docs/agent/event-handlers.html) for more info
* [Reproducing the environment in which monit runs](http://stackoverflow.com/questions/3356476/debugging-monit):

    ```
    # monit runs as superuser
    $ sudo su

    # the -i option ignores the inherited environment
    # this PATH is what monit supplies by default
    $ env -i PATH=/bin:/usr/bin:/sbin:/usr/sbin /bin/sh

    # su to the user under "as uid <username>"

    # try running start/stop program here
    $
    ```
* Travis' container-infra images can all be found [on
    quay](https://quay.io/organization/travisci)
* The `build-essential` apt package contains many things you typically need to
    compile code, such as `g++`, `gcc`, `make`, etc.
* If I won't have time to make dinner in the evening, working from home and
    making something that requires little babysitting is a convenient
    alternative
* `initctl` controls things managed by `upstart`. `upstart` conf files go into
    `/etc/init`.
* `mdfind`: an alwayss up to date version of `locate`, used by OSX's Spotlight
* In YAML, `~` means "nil".
* In yaml, to have a "verbatim" node (one where the information is passed
    through withou re-interpretation, except to remove the indentation':

    ```
    foo: |
      stompy
      this is
      grumpy time
    ```
* When someone disagrees with you in an unhelpful way, if you have the time:
    * think about what they are saying. If they said it another way, would you
        agree with them?
    * write down your thoughts about what makes your position good. write down
        thoughts about why their position is good. If your position is tied to
        work that you have done, at least your work won't go to waste. Everyone
        can learn from the work that you have done.
    * try to approach the person and give them helpful feedback about why their
        feedback is unhelpful.
* When you feel intimidated about having a conversation about something with
    another team, try picking the brains of some of the people on that team
    first. Through that conversation, you will likely become less ignorant, gain
    more empathy, learn more context, hear the biggest holes in your ideas...
* while using the `heroku/ruby` docker image, if you find that bundler is not
    finding the gems installed via `bundle install`, it could be that your local
    `.bundle` is overriding the one createed by the build step of `heroku/ruby`.
    `heroku/ruby` installs to `/app/heroku/ruby/bundle`, and for `bundler` to pick
    that up the `.bundle` file needs to be updated with that path. Example error:

    ```
    Trigger 5, RUN bundle exec rake assets:precompile
    Step 0 : RUN bundle exec rake assets:precompile
     ---> Running in 06e2edba3727
    /app/heroku/ruby/bundle/ruby/2.2.0/gems/bundler-1.9.10/lib/bundler/spec_set.rb:92:in `block in materialize': Could not find addressable-2.3.8 in any of the sources (Bundler::GemNotFound)
    ```
    Remove your local `.bundle` and things should work.
* Visual explanation of how raft works: http://thesecretlivesofdata.com/raft/
* Raft keeps a log of previous messages and a list of peers in `peers.json`. If
    you are just messing around and your cluster ends up in a weird state (such
    as old IP addresses showing up in the logs), you
    could delete all of raft's data (and serf's data, if you happen to be running
    consul -- see /var/consul/{serf,raft})
* If `service xyz restart` works on your server but there's no
    `/etc/init.d/xyz`, it may be run by [upstart](http://upstart.ubuntu.com/).
    Look for the config file in `/etc/init/xyz.conf`.
* Beware while playing with Consul if you lose cluster quorum or shut down your
    cluster! Consul requires manual intervention in this case in order to
    recover. If you don't manually intervene, it will be unable to elect a new
    leader. See [the outage recovery
    guide](https://consul.io/docs/guides/outage.html) and
    [numerous](https://github.com/hashicorp/consul/issues/993)
    [issues](https://github.com/hashicorp/consul/issues/454) on the
    [subject](https://github.com/hashicorp/consul/issues/924)
* In slack, option-click on a message's date will mark messages unread from
    there.
* `capybara-screenshot` is a gem you can include to take screenshots of the
    browser during failing capybara tests.

    If you are using `Test::Unit` to run your capybara tests, you should have
    this in your `test/test_helper.rb`:

    ```ruby
    require 'capybara-screenshot/testunit'
    ```
* In the webkit driver for capybara, `page.driver.error_messages` and
    `page.driver.console_messages` contain the errors and logs written to the
    console via javascript.
* `xvfb-run` lets you run a command inside a virtual X server. You can use it
    to run your webkit tests. E.g., `bundle exec xvfb-run ruby test`
* In travis, the `addons/hosts` is a list of hostnames pointing to localhost.
* One way to be able to restart a service over ssh without sudo (a bit of a
    roundabout and non-obvious way, but a way nonetheless) is to use `monit` to
    monitor a `restart.txt` file, which can be touched over ssh, and which
    triggers a restart. Mimicking `Passenger`.
* Passenger apps can be restarted by running `touch tmp/restart.txt` in the
    project directory.
* Many logging frameworks in java support the following for logging an
    exception:

    ```
    LOGGER.error("there was an error", exception);
    ```
* In GoCD, if you "rebuild" a failed build from the page containing the failed
    build, it will re-use the same config and everything (I think). So if you
    change the config (eg, you change the directory that a material should be
    checked out in), those changes will not affect the rebuilding of that build.
    BUT if you "rebuild" from the pipelines list page using the play button, it
    will apply the new config.
* `jar -tf` for listing the contents of a jar
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
* SOP means "standard operation procedure"
* To remove the "Apps" button from your chrome bookmark bar: right click on the
    bar and click "Show Apps Shortcut"
* [vagrant share](https://docs.vagrantup.com/v2/share/index.html) exposes your
    vagrant ports to the outside internet.
* There's an anchor button on 1password menu-bar-drop-down-windows that lets
    you anchor the window on top of everything even after the 1password menu has
    been dismissed. Useful for when you know you're going to need that same
    password again and again.
* Authentication problems via curl? Is it because you have 2-factor authentication
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
* If your keyboard is acting like it has a stuck key, but unsticking all keys
  doesn't fix it, nor does it work on your laptop, check that you don't have a
  3rd keyboard plugged in with something on top of it.
* Vagrant plugins installed via `vagrant plugin install <name>` are published
  as gems in the official rubygems repo
* [A list of vagrant boxes, complete with their download
  url](http://www.vagrantbox.es/)
* To run redis as installed by brew on Mac OSX, `redis-server
  /usr/local/etc/redis.conf`
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
