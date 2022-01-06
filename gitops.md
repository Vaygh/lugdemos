# GitOps for Fun and Profit

Presentation and demo for the Rock River Linux User Group - 13 January 2022

## What is GitOps?

The Cloud Native Computing Foundation [GitOps Working Group](https://opengitops.dev/) has a brief declaration of [GitOps Principles](https://github.com/open-gitops/documents/blob/v1.0.0/PRINCIPLES.md). At a high level, it says that system state should be stored in an immutable, version-controlled, declarative way. An agent will read these state declarations automatically, compare them to the running state, and attempt to bring the managed system in line with the desired state.

Obviously, as the name implies, there's a bias toward Git as the mechanism to achieve immutability and revision history, but it's not an absolute requirement. Theoretically, any reasonable version control system would be in line with these principles. Likewise, while there's a bias toward containerization and cloud-native systems to be managed via GitOps, they're not the only systems that can be managed this way. So long as an agent exists that can continuously monitor the system state, pull desired, declarative state definitions from the version control system, and attempt reconciliation, that system can be managed in a GitOps way.

## OK, So How's This Better Than Running Shell Scripts To Manage My Systems?

I'm glad you asked! :smile:

The short answer is that GitOps methodologies are more ***auditable***, ***scalable***, and ***reproducible*** than traditional system administration methods.

More ***auditable*** because the desired state of a given system, as well as the inventory of managed systems, is stored in a version control system repository. No more wondering what the `floopdy-schmoop-07` server is for: check the repo, look at the state declaration, and see what the system is supposed to be doing. Furthermore, you can browse through the commit history to see every time the desired state has been changed. No more wondering who changed the load balancer config to route production traffic to the dev servers: the answer is only a `git blame` away.

More ***scalable*** because state declarations can be easily applied to multiple systems. Need to create 150 new web servers to handle your Black Friday / Cyber Monday eCommerce surge? No problem. Bootstrap the base system, point the agent to your web server state declaration, and you're off to the races. Need to mitigate a [certain logging vulnerability](https://log4jmemes.com/) on all those systems? Again, no problem. Update the state declaration each time a new version of the library is released, and the agent will bring all the systems to the desired state. Unlike you, the agent never gets tired of reconciling system state, no matter how many times patches are released.

More ***reproducible*** because the desired system state is stored declaratively, separate from the system, and the operations needed to bring the system to that state are performed by a software agent. If `floopdy-schmoop-07` has an irrecoverable failure, replacing it is as simple as spinning up a new base system and connecting the reconciliation agent to it. From there, the agent will bring the system to the desired state. `floopdy-schmoop-07` is dead: long live `floopdy-schmoop-07`! You no longer have to worry if your shell script will work on the new system, or if some manually-modified config was copied over. You declare the desired state and the agent makes it happen.

## No System Is Perfect: What's The Catch?

1. Secrets: If all your system state declarations are in a code repository, how do you keep sensitive configs like passwords and connection information secret? Start by assuming that the repo itself is public (even if it's not, to the best of your knowledge) and **never** put secret information in the repository in plain text. Instead, leverage encryption to protect secrets before they're committed to the repository. Treat the encryption keys for these secrets as precious, mission-critical data, because they are. Tools such as Bitnami's [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) controller, AWS [Secrets Manager](https://aws.amazon.com/secrets-manager/), and Ansible [Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) are among a few that can help protect secrets when working in a GitOps way.

2. Migration: Moving dozens of systems with handcrafted config files and custom setup scripts into a GitOps framework may be difficult, if not impossible. There might not be an agent that can perform state reconciliation on a legacy system. You may not even have a full inventory of all the systems in the environment. You have to make a business decision as to whether the long-term benefits of GitOps methodologies are worth the pain of migrating older systems. The decision becomes much easier in a greenfield situation, such as a cloud migration, as a considerable amount of work must already be performed to succeed. The additional incremental work of adopting GitOps methodologies may be negligible (and might, in fact, make the migration easier.) You may decide - instead of back-porting legacy systems into your GitOps framework - to set a future-forward date: all systems created after X date will be managed via GitOps and legacy systems will be handled by legacy frameworks until they're retired.

3. Paradigm Shifts: Breaking old habits is hard. Adopting new tooling is hard. Under the GitOps paradigm, you won't be logging in to servers via SSH to update configurations. You won't be running `apt-get install apache2` any time soon. Instead, you'll be `git commit`ting lots of YAML files to a repo and watching a CD tool update systems. It's a different way to work, and it takes some getting used to, no doubt about it.

4. No Guarantees: The agent *attempts* reconciliation of the managed system with the desired state: there's no guarantee the reconciliation will be successful. You may have defined a desired state that is invalid, the agent could lose connectivity to the version control system, a cosmic ray could flip a bit; always have a "break glass in case of emergency" way to ensure you can manage the system manually.

## I'll Believe It When I See It (Demo Time!)

All right, let's deploy [searx](https://searx.github.io/searx/) to a [Kubernetes](https://kubernetes.io/) cluster via GitOps using [Argo CD](https://argo-cd.readthedocs.io/en/stable/).
