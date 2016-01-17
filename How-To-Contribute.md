**! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! !**

**-->    This page is out of date. Please check [here](https://github.com/openhab/openhab-distro/blob/master/CONTRIBUTING.md) for the latest info.**

**! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! ! !**


## Introduction

As for almost all open-source projects, contributions to the project are always welcome! However if you are interested in doing a contribution - be it a simple bug report or a big chunk of code for a new feature - it is good practise to adhere to the following workflow:

1. ask the [openHAB Community](https://community.openhab.org/) about a certain bug, behaviour, idea or missing feature.
1. discuss and refine the Requirement/Bug (if any) with the Community.
1. create an Issue in the [Github Issuetracker](https://github.com/openhab/openhab/issues) or contribute the necessary code changes through a [pull request](https://help.github.com/articles/creating-a-pull-request/) with reference to the discussion on the Group.
1. keep track of the created Issue (even if it doesn't apply anymore, etc.).

## A Note on openHAB 2 Compatibility

Please note that the core runtime and the designer are considered to be frozen in openHAB 1.x - only critical bug fixes should be contributed to it. The code of the core runtime and the designer is now actively maintained and heavily extended at [Eclipse SmartHome](https://github.com/eclipse/smarthome), where [it has moved in 2014](http://kaikreuzer.blogspot.de/2014/06/openhab-20-and-eclipse-smarthome.html) already and which is the base of [openHAB 2](https://github.com/openhab/openhab2). So if there is the need to change anything in the 1.x repository, please make sure that the same change is also contributed to the Eclipse SmartHome repository.
With regards to the addons (like bindings, actions, persistence services, etc.), openHAB 2 comes with a compatibility layer, which should make it possible to run 1.x addons on the 2.x runtime. All new addons that are contributed as a PR should also tested on openHAB 2 - once you have done that, you should create a [PR like this one](https://github.com/openhab/openhab2/pull/282/files) against openHAB 2. This is a mandatory prerequisite for having the contribution merged in openHAB 1.x. If you have any questions or concerns about this, please feel free to ask on the mailing list.

The remainder of this page will give you the details of how to contribute more specifically:

## Best Practices for Contributors

The simplest way of contributing is probably to report bugs. You can do so using the [Issue Tracker](https://github.com/openhab/openhab/issues?state=open). 

If you are in doubt whether it is a bug or not, you can also first refer to [the forum](https://community.openhab.org) instead of entering a bug report.

The same is true if you intend to implement/contribute a new feature. Please always first discuss your idea in the group as this will ensure a clear project direction and avoid the situation that different people unknowingly work on the same feature.

## Code Handling

To make code changes to the openHAB code base yourself, all you have to do is to create a local clone of the repository (git clone https://github.com/openhab/openhab/). See [IDE Setup](IDE-Setup) on how to set up a development environment and read [How to implement a binding](How-To-Implement-A-Binding) if you plan to implement a new binding.

If you want to contribute your code or just want to share it with others, you can [create a fork of the official repository](https://github.com/openhab/openhab/fork) at any time, for which you will have full access so that your local changesets can be pushed to it.

Once your code is ready and accepted (see code style section below), it is then easy for the project owners to pull your changesets into the official repository - all you have to do is to [create a pull request](https://help.github.com/articles/creating-a-pull-request).

**Please note:** Your pull request should contain only one single commit before merging. You can achieve that by [squashing your commits into one](https://github.com/ginatrapani/todo.txt-android/wiki/Squash-All-Commits-Related-to-a-Single-Issue-into-a-Single-Commit).

## Licensing

As openHAB is licensed under the Eclipse Public License (EPL), your code should include the [standard openHAB license headers](https://github.com/openhab/openhab/blob/master/src/etc/header.txt) as well. To automatically add it to your code, you can run `mvn license:format -Dtycho.mode=maven`.

By contributing code to openHAB, we therefore implicitly assume your approval to make it available under the EPL and that you have the right to give us the approval (i.e. the code does not contain any intellectual property that belongs to somebody else).
By attaching code to the issue tracker or posting code in the discussion groups, the contributor implicitly grants rights to use the code under the above mentioned terms and conditions. 

## Code Style & Conventions

To ensure code quality in our official repository, the project owners do code reviews before merging contributions into the main repository. There are some rules that every contribution should follow:

1. The [Java naming conventions](http://java.about.com/od/javasyntax/a/nameconventions.htm) should be used.
1. Every Java file must have a [license header](https://github.com/openhab/openhab/blob/master/src/etc/header.txt). You can run ```mvn license:format -Dtycho.mode=maven``` on the root of the repo to automatically add missing headers.
1. Every class, interface and enumeration should have JavaDoc describing its purpose and usage.
1. Every class, interface and enumeration must have an @author tag in its JavaDoc for every author that wrote a substantial part of the file.
1. Every constant, field and method with default, protected or public visibility should have JavaDoc (optional, but encouraged for private visibility as well)
1. Code must be formatted using the provided code formatter and clean up settings (import them into your IDE).

## OSGi Bundles

1. Every bundle must contain a Maven pom.xml with a version and artifact name that is in sync with the manifest entry. The pom.xml must reference the correct parent pom (which is usually in the parent folder).
1. Every bundle must contain an about.html file, providing license information.
1. Every bundle must contain a build.properties file, which lists all resources that should end up in the binary under bin.includes.
1. The manifest must not contain any "Require-Bundle" entries. Instead, "Import-Package" must be used.
1. The manifest must not export any internal package
1. The manifest must not have any version constraint on package imports, unless this is thoughtfully added. Note that Eclipse automatically adds these constraints based on the version in the target platform, which might be too high in many cases.
1. The manifest must include all services in the Service-Component entry. A good approach is to put OSGI-INF/*.xml in there.

## Runtime Behavior

1. Overridden methods from abstract classes or interfaces are expected to return fast unless otherwise stated in their JavaDoc. Expensive operations should therefore rather be scheduled as a job.
1. Creation of threads must be avoided. Instead, resort into using existing schedulers which use pre-configured thread pools. If there is no suitable scheduler available, start a discussion in the forum about it rather than creating a thread by yourself.
1. Bundles need to cleanly start and stop without throwing exceptions or malfunctioning. This can be tested by manually starting and stopping the bundle from the console (stop <bundle-id> resp. start <bundle-id>).
1. Bundles must not require any substantial CPU time. Test this e.g. using "top" or VisualVM and compare CPU utilization with your bundle stopped vs. started.