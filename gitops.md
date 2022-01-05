# GitOps for Fun and Profit

Presentation and demo for the Rock River Linux User Group - 13 January 2022

## What is GitOps?

The Cloud Native Computing Foundation [GitOps Working Group](https://opengitops.dev/) has a brief declaration of [GitOps Principles](https://github.com/open-gitops/documents/blob/v1.0.0/PRINCIPLES.md). At a high level, it says that system state should be stored in an immutable, version-controlled, declarative way. An agent will read these state declarations automatically, compare them to the running state, and attempt to bring the managed system in line with the desired state.

Obviously, as the name implies, there's a bias toward Git as the mechanism to achieve immutability and revision history, but it's not an absolute requirement. Theoretically, any reasonable version control system would be in line with these principles. Likewise, while there's a bias toward containerization and cloud-native systems to be managed via GitOps, they're not the only systems that can be managed this way. So long as an agent exists that can continuously monitor the system state, pull desired, declarative state from the version control system, and attempt reconciliation, that system can be managed in a GitOps way.

## OK, So How's This Better Than Running Shell Scripts To Manage My Systems?

I'm glad you asked! :)

The short answer is that GitOps methodologies are more auditable, scalable, and reproducible than traditional system administration methods.

More auditable because the desired state of a given system, as well as the inventory of managed systems, is stored in a VCS repository. No more wondering what the `floopdy-schmoop-07` server is for: check the repo, look at the state declaration, see what the system is supposed to be doing. Furthermore, you can browse through the commit history to see every time the desired state has been changed. No more wondering who changed the load balancer config to route production traffic to the dev servers: the answer is only a `git blame` away.

More scalable because

More reproducible because the state declaration is stored separate from the system and the operations needed to bring the system to that state are performed by a software agent. If `floopdy-schmoop-07` has an irrecoverable failure, replacing it is as simple as spinning up a new base system and connecting the reconciliation agent to it. From there, the agent will bring the system to the desired state. `floopdy-schmoop-07` is dead: long live `floopdy-schmoop-07`! You no longer have to worry if your shell script will work on the new system, or if some manually-modified config was copied over. You declare the desired state and the agent makes it happen.

## No System Is Perfect: What's The Catch?

secrets
migrating existing systems: legacy and unknowns
paradigm shift, new tooling

## Sounds Too Good To Be True: I'll Believe It When I See It

All right, let's deploy [searx](https://searx.github.io/searx/) to a [Kubernetes](https://kubernetes.io/) cluster via GitOps using [Argo CD](https://argo-cd.readthedocs.io/en/stable/).
