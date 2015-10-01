# Tools

- vagrant-proxyconf
- envconsul
- consulate
- nsq
- virtualbox and vagrant in a container
- vagrant-spec


# The tao of hashicorp

workflows over technology. Nail the workflow, then start thinking about the
technologies you will integrate with.

Simple, modular and composable. Good for understanding the boundaries of tools.
isolates problems, good for auditing.

communicating sequential processes - processes should be able to communicate
status outside of it using the network, but not be bound to the things they are
communicating with.

immutability. Be able to say x is a version of something, you can roll forward
/backward. Not one big immutable thing, but think about layers of immutability.
eg the ami, and the application layer (maybe scheduled by nomad).

Versioning through codification. before codification, we had an oral tradition
in ops! A very lossy method of education. This paired with immutability and
version control allows people to see how things have changed over time.

Automation through codificaion. There's bash scripts, and then there are higher
levels of abstractions.

resilient systems. Infrastructures need to handle failure better as they grow.
when you are operating in a CSP environment, need to handle unexpected input
gracefully. Terraform is a great example of this - we knew cloud providers can
be unreliable at times, we built partial state in terraform, to see what had
been done and where it needs to go, to track partially provisioned things.

pragmatism - we have principles, but we are not dogmatic about them. If the
most pragmatic solution is not to be resilient, don't do it. it's not about
compromise, but it's about being open minded.
