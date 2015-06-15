* When you first create a pipeline, you will need to "unpause" it before it'll
    run
* You cannot rename a pipeline in gocd without losing all its history
* When you create a new worker, need to "authorize" it
    * [from the gocd interface](http://www.go.cd/documentation/user/current/configuration/managing_a_build_cloud.html),
      or
    * [by assigning it a registration token](http://www.go.cd/documentation/user/current/advanced_usage/agent_auto_register.html)
* To configure the log level for GoCD, edit `/etc/go/log4j.properties`
