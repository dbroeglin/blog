---
title: "One year of PowerShell or DevOps in the Microsoft world..."
date: 2016-12-01
draft: true # Do not publish as it was published elsewhere
---

Holidays are a time for reflection and giving back. This year, I thought I would reflect on my ongoing Linux to Microsoft transition and give back a few tidbits I learned about DevOps in the Microsoft world. This article is meant to illustrate a journey from Linux to Microsoft in hope that the “from the trenches” perspective will help others attempting the same transition.

Let's start with a quick background to set the stage.

# Background

After an education in both software engineering and distributed systems I worked for several IT companies, small and big, in a variety of domains such as telcos, education, justice, transportation, luxury goods or the press. I was lucky enough to have the opportunity to work with bright people very early in my career. I'm very grateful to all of them for I learned a lot from them and was spared quite a few mistakes. One of the subjects I was introduced to early is automation. Both through my co-workers and through seminal books like _[The Pragmatic Programmer: From Journeyman to Master][prag_prog]_ and its sequel _[Pragmatic Project Automation][prag_automation]_. Which in turn pushed me towards automated configuration management in the early days.

If memory serves, I discovered Linux somewhere around 1994 and was instantly hooked. I still recall that first Slackware distribution with its 50 something floppy disks. For twenty years I've worked mainly on Linux and Unixes trying studiously to avoid any Microsoft product. Some people, may even recall me swearing I would never work on Windows. However, a few years ago Microsoft changed its approach and I was, coincidentally, presented opportunities to work in Microsoft environments. As was once said by John H. Patterson, “Only fools and dead men don't change their minds”. So I went on and changed mine, one step at a time...

So, armed with almost 20 years of Linux experience and 10 years of automated configuration management, I jumped into Microsoft automated configuration management. 

# Step 1: Develop on Windows, Deploy on Linux

The first step was a Java project that was starting. In that early design and setup phase, only a few people were involved. I was brought in to help with Java architecture and software factory setup. Having setup quite a few factories already that part was not new to me. However, we would have to onboard a sizable number of people in a small amount of time. That would be a challenge. Because, at the time it took quite a bit of time to setup a working environment for a developer, tester, architect or DBA. 

During previous projects, I had used [Vagrant][vagrant] to setup development environments that could be shared easily. Vagrant is basically a wrapper around a virtualization tool (at the time VirtualBox, but in the meantime Hyper-V and VMWare were added too) that helps in setting up a virtual machine. It starts from a fixed _base_ image, but allows for additional setup instructions to be declared in a file that can be shared in the project's source control repository. Everything needed to setup the environment is declared in a file called `Vagrantfile` and when a developer wishes to setup his he just executes `vagrant up`.

A few minutes later, a fully operational environment is running inside a virtual machine which in turn runs on his desktop environment. In a MacOSX or Linux environment, the developer usually edits files by running his editor on the host system but builds, deploys and tests the system in the virtual machine. Moreover, Vagrant allows for provisioning scripts declared in the `Vagrantfile` but it also allows integrating with a [Puppet][puppet] or [Chef][chef] infrastructure. Which means that the development virtual machine can be provisioned exactly like the _test_ and _production_ environments. 

It also allows for a nice DevOps pipeline by permitting each developer to experiment with an infrastructure change on his own virtual machine. When satisfied she can then share that change through source control. Her fellow developers will benefit of the change by refreshing their sources and executing a simple `vagrant provision` command. The change can then be merged, again through source control, in the _test_ and _production_ environments. Developers and operation people can experiment in their environment without fear and at a very low cost. Breaking their environment would only mean that they have to revert their changes through source control and execute a `vagrant destroy; vagrant up` command (with the additional benefit that they get to grab a cup of coffee while this is running). 

However, after a few experimentation with Vagrant on Windows we quickly found out that the developer experience was not satisfactory. Performance was very degraded, editing on Windows while building on Linux proved to be quite challenging. On the other hand, the JEE technology with thousands of java, XML and ZIP files was posing a greater challenge to virtualized environment that the more usual Rails, Node or Python environments.

As we already planned on using Puppet for test and production environment deployment, we settled on deploying each developer's desktop environment with the same tool. This allowed us to almost completely automate desktop environment setup. It had a few drawbacks though. Mainly, for those project members that worked on their laptop and had previously installed tooling, the setup process might interfere with those tools or fail altogether because they were set up in an unexpected way. We also had the usual issues that were found out only in the _test_ environment because developers tested on their own Windows environment.

# Step 2: Deploy on windows in a Linux shop

The second step was a bit different. This time, we were both developing and deploying on Windows. The project was only a few months old, the deployment process was manual and configuration management was starting to become an issue. Most of the rest of the company was deploying applications on Linux and the infrastructure team had already introduced [Puppet][puppet] to manage those hosts. We ourselves used [Puppet][puppet] to deploy our hosts to build on existing experience and to prepare for an eventual migration from Windows to Linux.

All environments were managed by the same system which allowed developers to have local setups almost identical to _test_ and _production_ environments. Deployments and configuration changes were propagated along the pipeline by successively merging from _development_ to _test_ and from _test_ to _production_. Configuration data was stored in [HieraDB][hieradb] a part of Puppet that provides a hierarchical configuration database. 

This particular software, we were developing, required complicated configuration management and we rapidly reaped the benefits of introducing automation. Especially when deployments started multiplying.
However, it revealed much more vividly than in step 1 that Puppet, or any tool that grew up in the Linux world, has an impedance mismatch when deployed on Windows. Basically, Unixes are based on files and executables whereas Windows is much more complex. The registry, the right management system, drivers, host processes, services, etc. All those concepts are not hugely different from Linux equivalents. But nonetheless, different enough to make things complicated with tools like [Puppet][puppet], [Chef][chef], [Ansible][ansible], [Salt][salt], etc. Nothing is completely impossible, but you have to work harder to achieve the same result on Windows than in their native OSes. 

Especially, when you cannot handle the task with pre-existing _resources_ and need to add some new behavior to the configuration management tool. This can be achieved either by extending them with Ruby code for [Puppet][puppet] and [Chef][chef] or Python code for [Ansible][ansible] and [Salt][salt]. But even general purpose languages like Ruby and Python that are ported to Windows show some of that impedance mismatch. Additionally, most of the time, they are not known by existing IT professional. We chose to use [Bash][bash] through the [MSysGit][msysgit] toolbox because both developers and IT professionals were familiar with it. However, that still was not satisfactory. Even if that version of [Bash][bash] is very well integrated with Windows, it still does not work seamlessly. Very simple tasks like creating a properly configured Windows service required juggling with different concepts and commands.

Note: in the meantime Microsoft has released a beta version of _[Bash on Windows][bash_on_windows]_ which is another version of [Bash][bash] running on Windows. We did not use it as much as the [MSysGit][msysgit] version but it seems to exhibit the same inherent difficulties. 

# Step 3: What the heck, let's learn PowerShell

The latest step of our journey occurred in a full Microsoft environment. Development and deployment happened almost exclusively on Windows. Modern configuration management like [Puppet][puppet] or [Chef][chef] was not yet introduced. However, several efforts were made to introduce automation, but mostly of the _mechanization_ sort (see below). Repetitive tasks have been accumulated together into scripts, in some cases those scripts are called through _workflow like_ graphical interfaces, but mostly, the control and knowledge remained with the IT professional.

This time around, people already were familiar with [PowerShell][powershell], the scripting language introduced by Microsoft in 2006. The approach we took was to expand on that existing body of knowledge with extensive training and systematic exploration of PowerShell solutions to our issues _before_ even considering more advanced but also more alien solutions. We found that [PowerShell][powershell] provided most of the required building blocks for a full deployment and configuration management stack.

## PowerShell 

[PowerShell][powershell] is a scripting language introduced by Microsoft in 2006. But it was already laid out in the [Monad Manifesto][monad_manifesto] by Jeffrey Snover in 2002. As explained in the manifesto, PowerShell takes a new approach to old issues. It is a shell with a pipeline. However, PowerShell passes objects down the pipeline, not text or binary data like its Unix relatives. Which saves quite a bit of work when composing simpler commands together. 

PowerShell also leverages the .Net framework which makes it as powerful as any  general purpose programming language. Access to the .Net framework means that any API available to .Net programmers will also be accessible through PowerShell. But it does not stop there. It also provides [jobs][ps_jobs], [remoting][ps_remoting], [workflows][ps_workflows], [package managment][ps_packages] and many other features. Among those, two are of particular interest when building an automation solution: _Desired State Configuration_ and _Just Enough Administration_.

## Desired State Configuration

[DSC][dsc_overview] builds on top of PowerShell to implement configuration management building blocks called _resources_. Those resources, are very similar to [Puppet][puppet] resources and similarly allow IT professionals to _declaratively_ express what the state of the system should be. The actual _imperative_ implementation that changes the system to the desired state is implemented by a PowerShell module. 

DSC also provides a declarative [domain specific language][dsl] to express a system's configuration and an agent that can apply that configuration to a specific host. The agent is an integral part of the Windows Management Framework which means that it will be deployed everywhere PowerShell is.

Transforming traditional imperative scripts into DSC resources allows us to achieve [idempotence][idempotence].  Which in turn allows us to handle any configuration change like a simple change in the declared configuration. DSC will then ensure the system ends in the required state from whatever previous state it was in. Even, a manual change, of the system would be caught by DSC as a configuration drift and could be automatically corrected, depending on the agent's configuration.

DSC can do much more, however, it is not as mature a solutions as [Puppet][puppet] or [Chef][chef]. It lacks the ecosystem around it: a configuration database, reporting tools, easy but powerful configuration composition and reuse, etc. Which is why our current approach is to associate both tools. The impedance mismatch mentioned earlier can be solved by letting Puppet or Chef use DSC resources to effect configuration changes on Windows. This is a win/win situation. Benefit from all the power behind the Puppet and Chef ecosystems, while still leveraging PowerShell, which is native to Windows and already known to IT professionals.

Our current preference is to use Chef. Chef is a bit less mature than Puppet but it seems that Chef and Microsoft [actively work together][chef_windows] to integrate both solutions.

## Just Enough Administration

[JEA][jea] is a tool that helps reduce administrative rights dissemination. Lots of operations require administrative or at least somehow elevated rights to be performed. That usually means that if you have to perform one of those operation (even if just is a read-only operation) you would get administrative rights. Or some ad hoc solution would be put in place to somehow allow you to do what you needed to. This can lead to pretty complex and fragile solutions. 

From my Linux background point of view, JEA is like `sudo` on steroids. It handles both privilege elevation, RBAC, remoting and integrates nicely with PowerShell scripting. It can easily be deployed via DSC which helps ensuring limited and uniform access rights throughout the whole system.

## Perspective

That latest step is still quite new and, for some parts, a work in progress. We have solved all the issues encountered in previous experiences and even a few new ones. Which leads us to think we are on the right track. Moreover, it seems the vision we have created aligns nicely with the “new” Microsoft. For instance, the recent open sourcing of .Net and PowerShell, while at the same time porting it to Linux and MacOSX, opens new avenues for automating our few existing Linux systems. 

# Lessons learned

The following lessons were learned the hard way. They are a bit opinionated. But, all are rooted in real life situations where not following them meant failure...

## Start small.

In my experience, trying to automate the whole system at once ends badly. At best, only those part of the system that were _easy_ end up automated, leaving huge gaps where manual intervention is still needed. At worst, the effort fails completely.  

Start small by considering only a single part of the system. However, ensure that that part is fully automated. Do not leave some manual steps in-between automated parts. The automation should encompass all use cases, even those considered _exceptions_. If it looks like they are too different to be automated, it usually means that the automation system is too rigid, too specialized to some local or current way of doing things. Not being able to handle today's exceptions is a good indicator that the system we are creating will probably not be able to handle future either. Also, partial coverage, means that we cannot have full confidence in our ability to reconstruct the system. Be it because we are facing a major disaster or, simply, because we need to experiment with a copy of the actual system.

This makes for slower progress, but allows you to get a solid foothold on the automated ground. Expanding from that solid ground will be much easier in the future. 

## "Mechanization" is not automation...

In my experience, in environments where the concept of _automation_ is new, it is often confused with _mechanization_. It is what might be called the _reduce 10 steps to 1_ syndrome.

Wikipedia defines [Mechanization][mechanization] as:

> Mechanization is the process of changing from working largely or exclusively by hand or with animals to doing that work with machinery.

Too often, automation is confused with mechanization. Allowing developers to submit an SQL schema migration through a form is a form of mechanization. The developer does not need to open the a SQL tool, enter the proper credentials and execute the SQL file by hand. However, it is not automation yet. The process is still almost exclusively under the control of human being (animal labor is now mostly eliminated from our IT processes, although we still occasionally see some developers [talking to ducks]). Wikipedia defines [Automation][automation] as:

> Automation is the use of various control systems for operating equipment [...] with minimal or reduced human intervention. 

To really automate our SQL database schema migrations, human intervention should be eliminated altogether. Which means, that the developer should only specify which version of the database schema is required and automation will take care of bringing the database to the proper state. Tools like [Liquibase][liquibase] or [Flyway][flyway] can help a lot with that. To the point that humans do not even need to ask for a database schema version change. The application can, upon starting, check that the database is in the proper state and, if not, apply the relevant migrations.

While introducing automation instead of mechanization is a bit harder it has tremendous advantages down the road. To quote Wikipedia one last time:

> The biggest benefit of automation is that it saves labor; however, it is also used to save energy and materials and to improve quality, accuracy and precision.

Automation completely eliminates human error by eliminating human decision from the process. Which in turn improves quality and repeatability.

## Keep away from shiny tools

With the introduction of _Infrastructure as Code_ IT professionals became de facto developers. That means their activity has changed from simply deploying and operating software to actually building the software that actually deploys that software. Even when that software is actually some piece of infrastructure.

That poses a challenge to any IT professional that did not start his career as a developer. However, in the Linux world any IT professional has been exposed to a variety of scripting languages which performed a lot of tasks from the most mundane to some quite sophisticated (for instance Gentoo's build system, [ebuild][ebuild]). The Microsoft world however has long relied on graphical user interfaces and manual operations. It is thus natural to turn to graphical tools that promise to simplify infrastructure as code by allowing building complex automation through graphical manipulation. I have yet to find a graphical tool that actually fulfills that promise. During the last years I found quite a few instances where they made things way worse.

Coding is coding. To this day, most of coding occurs by _writing_ code. Yes, there are a few specialized tasks that are better handled by graphical tools, but general coding tasks usually involve editing some kind of text. So, even if you can leverage graphical tools for some of your automation tasks, in the end, a general purpose scripting language will always be necessary.

The language of choice, on Microsoft Windows, is PowerShell. It is still a Shell with its roots firmly planted in the Unix world but is also very well integrated with the Windows system and Windows applications. All of which makes your life so much easier. If you only learned PowerShell by doing the same stuff you did with .BAT files and sprinkling a bit of Stack Overflow, try to get some formal [training][powershell_training]. It will be time and money well spent that will pay for itself time and time again.

## Involve IT professionals

Automation should be built by the people who are the most intimate with the system being automated. If the automation solution is built by outside people it often lacks the proper adoption that would permit it to grow. In some extreme cases, the knowledge of what the automation actually does is lost in the system. Developers move on to building other systems and IT professionals lack the expertise to actually dig into the system when needed. 

Of course, in most occurrences, IT professionals are not _yet_ developers and lack the set of skills required to build the automation tools. In which case, bringing in external resources to help mentor them in those new fields will speed things up and prevent basic mistakes. However, at the end of the process, IT professionals should be in charge and complete control of their tool chain.  

One way to flatten their learning curve while still leaving them in charge is to help them setup a software factory and software tooling very early in the process. If PowerShell module scaffolds can be generated, tests automatically executed, modules packaged and deployed automatically to the repository, etc. they can concentrate on what really matters: the automation system they are building.

# The release pipeline model

Earlier this year Microsoft published a white paper about the [Release Pipeline Model][release pipeline model] which neatly sums up everything we mentioned here (and more). I strongly encourage anyone attempting an automated deployment and configuration management effort in the Microsoft world to read it. Most of what you would need to start, but is not mentioned in this post, can be found in it.

# Looking forward

It is probably too soon to gain enough perspective on how automated configuration management will evolve in the Microsoft world. On the other hand, all pieces seem to fall neatly in place and work well together.  

[prag_prog]: https://en.wikipedia.org/wiki/The_Pragmatic_Programmer
[prag_automation]: https://pragprog.com/book/auto/pragmatic-project-automation
[vagrant]: https://www.vagrantup.com/about.html
[dsc_overview]: https://msdn.microsoft.com/en-us/powershell/dsc/overview
[release pipeline model]: https://msdn.microsoft.com/en-us/powershell/dsc/whitepapers
[mechanization]: https://en.wikipedia.org/wiki/Mechanization
[automation]: https://en.wikipedia.org/wiki/Automation
[talking to ducks]: https://en.wikipedia.org/wiki/Rubber_duck_debugging
[liquibase]: http://www.liquibase.org/
[flyway]: https://flywaydb.org/
[ebuild]: http://www.funtoo.org/Bash_by_Example,_Part_3
[powershell_training]: https://mva.microsoft.com/training-topics/powershell
[puppet]: https://puppet.com/
[chef]: https://www.chef.io/chef/
[ansible]: https://www.ansible.com/
[salt]: https://saltstack.com/
[hieradb]: https://docs.puppet.com/hiera/
[bash]: https://www.gnu.org/software/bash/
[msysgit]: https://github.com/msysgit/msysgit
[bash_on_windows]: https://msdn.microsoft.com/en-us/commandline/wsl/about
[powershell]: https://technet.microsoft.com/en-us/library/bb978526.aspx
[monad_manifesto]: http://www.jsnover.com/blog/2011/10/01/monad-manifesto/
[ps_jobs]: https://msdn.microsoft.com/en-us/library/dd878288(v=vs.85).aspx
[ps_remoting]: https://msdn.microsoft.com/en-us/powershell/scripting/core-powershell/running-remote-commands
[ps_workflows]: https://msdn.microsoft.com/en-us/library/jj148984(v=vs.85).aspx
[ps_packages]: https://msdn.microsoft.com/en-us/powershell/reference/5.0/microsoft.powershell.core/about/about_packagemanagement
[dsl]: https://en.wikipedia.org/wiki/Domain-specific_language
[idempotence]: https://en.wikipedia.org/wiki/Idempotence
[chef_windows]: https://channel9.msdn.com/Events/DevOps-Microsoft-Chef/ChefConf-2016/Chef-and-PowerShell-DSC-Bringing-Your-Machine-to-Its-Desired-State
[jea]: https://msdn.microsoft.com/en-us/library/dn896648.aspx