Things to read
---------

* bridget kromhout (?)
* Ops pain thermodynamics
* The Phoenix Project

Hacking the CxO code / J. Paul Reed @SoberBuildEng
---------

* CapEx vs OpEx budget - infrastructure as code means that previously opEx items
  can come out of the capEx budget
* Know the sources & cycles of the money that funds the projects you care about.
  Ask to do experiments when the new budget comes out, not at the tail end of
  the previous budget.
* Convince by talking about potential future state "This is where we could be".
  Tell a story.
* Success of your project means that *you* are not necessary for the success of
  your project. It can't be your little baby that only you know how to use.
* What does success of your project mean to the CTO? Use that in your story to
  convince.

Agent of Change - Here's Your Helmet / H. "Waldo" Gunenwald @gwaldo
----------

Are you looking for a dev-ops role? here are some things to think about.
* An agent of change is not a peer, it's a director. Don't be hired in devops as
  a peer, you need to be given the authority to change things.
* Prepare to have arguments as you challenge the status quo. Lots.
* Your expectations frame your outlook. At one job, we had a keurig and the
  expectation was that you cleaned up after yourself. Disobeying that rule
  caused lots of conflict. At another job, there was a keurig and it was
  people's expectation that they would clean up after the person before them,
  and it was totally fine.
* Authority vs Responsibility

Some questions to ask when you are interviewing:
* What does a postmortem look like?
* What tools are run in house vs SaaS?
* Are developers first to be paged?
* Are your ops teams more of the firefighting variety, or do they build things?
* How are you able to create change? where do you fit in the org chart? Is the
  org chart giant so that any change will require a lot of momentum?
* Do they want change, or are they hiring for a leaf node in the org? Are they
  hiring you as a consultant (who is payed to give advice) or as an employee?

Your fresh eyes are the superpower they hired you for.

Follow: @devopsjerk

Chef, test kitchen, Docker, CI. Oh My! / Arthur Maltson @amaltson
---------

Test-driven infrastructure

`kitchen` offers choices in the following: driver, communication, provisioner,
and testing platform. So you can tell it to run on docker images or AWS, you can
tell it to provision with chef or puppet, etc.

```yaml
driver:
  name: docker
```

```bash
kitchen login
kitchen converge
kitchen verify
```

Server spec:

```ruby
describe port(80) do
end

its(:exit_status)
its(:stdout)
```

Every command that you would use to verify that your service is running, like
`nc -vz localhost 8080`, turn into an equivalent test.

`chef-audit`: run server spec tests in production

Open Space: Remote teams
-----------

* `zoom`? be willing to show your face in remote meetings. It helps eliminate
  "these guys vs those guys" mentality, and commitment means more.
* Spend $250 on a good microphone and good camera. Don't use a laptop pointed at
  a barely visible whiteboard such that the mic is pointed away from everyone.
* If one person on your team is remote, the whole team should work as though
  they are remote.
* In remote teams, you can't rely on shadowing as an onboarding strategy.
* If some of the team is collocated, leverage a central team portal open to
  other locations to expose "technical chatter"
* You need to hire people who will naturally self-assemble, self-congregate, and
  communicate.

Open Space: Keeping on top of technology
---------

* Listen to podcasts sped up, 2x faster
* Take the bus rather than driving
* [Pocket Casts](https://play.google.com/store/apps/details?id=au.com.shiftyjelly.pocketcasts&hl=en)
  is a podcast app that has some nice features: rewinds 5s if you paused for
  1min, rewinds 1min if you paused for 24h
* Use twitter to gauge traction of blog posts. Don't read it the first time, but
  read it the second or third time it comes up in your feed
* Cloudflare's blog is really good if like stories of how they troubleshoot and
  solve problems
* LWN (Linux Weekly News) - a lot of this you can disregard if you're not a
  kernel developer, but often you can get ahead of the curve and hear about the
  future before it arrives. Stuff like "the new firewall was just merged" --
  Didn't even know there was a new firewall!
* Follow @martinfowler
* Follow @jeffbarr to hear about amazon stuff

Open Space: infrastructure testing
-------

* `chef-spec` is for testing in a pretend world. Kind of like unit tests. some
  people say it's not as useful for infrastructure testing.
* `server-spec`/`test kitchen` is for testing in an actual world. Like an
  integration test. More useful than unit testing.

Server spec is great for single node systems. But how do you test that X on node
Y can talk to A on node B? Test kitchen can converge multiple machines, and then
you can run your tests in that new environment.

chef-audit to test production servers

`blueprint`: reverse engineer a (manually) configured server to write
configuration automation

Open Space: managing docker containers
-----

How do you configure your docker containers per environment (like prod, qa,
dev)?

* Use etcd or zookeeper for configuration values
* build different images per environment using the same chef recipe
* What about using config management to write the files (to the host) needed for
  the container? This doesn't scale. Are you going to put those files on each
  node, even if they're not running that survice?
* Use environment variables - (IMO, this is a pretty rudimentary mechanism of
  solving this - all you have are strings. Are you going to write config files
  from those environment variables? are you going to have the app query the
  environment for those values?

Someone likes mesos. Marathon, not so much.

Docker-compose is nice, but it doesn't understand that a container needs to be
running before its dependent can run.

What about nginx in a container? How do you upgrade? You have to connection
drain on an instance, direct to other nginx, upgrade (or get rid of the node
altogether), then direct traffic back. This assumes that you have an ELB in
front of everything.

Someone else thinks that cloudfoundry solves the upgrade and environment
problems. Also, http://bosh.io/docs/ for 0-downtime upgrades.


Bulding Systems focussed on developer happiness / Philip Corliss @pcorliss
------

* Make it easier for users to do the right things than the wrong things

Other excerpts
-----

* Call skunkworks things and prototypes "experiments". Identify who the
  nay-sayers and supporters of each experiment, and be sure to keep both parties
  in the loop. If you can convince the nay-sayers, you will have made a great
  advocate.
