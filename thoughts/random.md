- All ideas are welcome, not all ideas are relevant
- Asking for a favour in an ecosystem where you don't know who can actually
    grant the favour:

    > Hi folks, who do I need to talk to to get setup with such and such
    > account?
- Just because other people have a lot of context on an issue,  don't shy away
    from the issue and assume that you can't understand it. Sometimes the issue
    ends up being simpler than you (or the others) imagined.
- I took a thought -- "yay my new earbuds arrived" -- and turned it into 3 great
    things: "Yay! Eric is here! he brought my new earbuds - the ones I
    ordered using the gift card that those people gave me." I became a lot
    happier, because I realized (a) Eric was here, (b) I had my headphones, and
    (c) I felt appreciated because the gift card was from some people who were
    thanking me for something.
- It is amazing how many problems I solve in the first hour of the day against
    which I have spent a large chunk of the day previous banging my head.
- One reason I think that chefspec tests are weird is that they are
    testing the implementation of a recipe. A good unit test for a class will
    only test the external interface of that class, whereas chefspec tests are
    testing the internal implementation. I guess that's an artifact of chef
    code modifying global state, and the external interface of a recipe is the
    same as the internal state.
- A good time to move from bash script to actual code: at the 50L mark. Same
    threshold can apply to breaking up large files or classes. (Courtesy
    @deyman)
- I should probably assume that chat, like all asyncronous channels, is faulty,
    and important messages should be retried if they have not been acknowledged.
- Things whose data sources are simple yaml files or hash tables are nice.
    Things that make it hard to programmatically update their data sources are
    not nice :(
- When proposing a change, phrase the proposition as "Does anyone object to
    this?" vs "Is this OK?". There are a lot of ways to do things and you work
    with smart people who have lots of strong, well-informed opinions. Avoid
    triggering that in the wrong way.
- When someone is explaining something you asked about, don't get annoyed when
    they explain something you already know. You've already decided it's ok to
    not know something by asking in the first place. So you should not feel
    ashamed or offended when someone thinks there's something else you don't
    know.
- "Do the simplest thing that could possibly work." - Ward Cunningham
- When you build a command line tool to do something, it means it can be run
    from jenkins :)
- When some code you write interacts with private github repos, you need to
    take care that the extra step of authentication doesn't make it impossible
    to automate your processes.
