<table><tr><td>:warning:</td>
<td>
<hr width="100%">
<b>
This page is out of date. Please check <a href="http://docs.openhab.org/developers/development/ide.html">here</a> for the latest info.
</b>
<hr>
</td>
<td>:warning:</td></tr></table>

How to set up a development environment for openHAB

## Introduction

If you are a developer and not a pure user yourself, you might want to setup your Eclipse IDE, so that you can debug and develop openHAB bundles yourself. There are (at least) two options for getting the IDE running: pure eclipse and vagrant.

## 'Pure Eclipse' Instructions

1. Create a local clone of the openHAB repository by running "git clone https://github.com/openhab/openhab" in a suitable folder.
1. Download and install oracle jdk 1.7
1. Download and install the [Eclipse for RCP and RAP Developers](https://www.eclipse.org/downloads/packages/eclipse-rcp-and-rap-developers/lunasr2).
1. Create a new workspace.
1. Choose `File` → `Import` → `General` → `Existing Projects into Workspace`, enter your clone repository directory as the root directory and press "Finish".
1. After the import is done, you have to select the target platform by selecting `Window` → `Preferences` → `Plug-in Development` → `Target Platform` → `openHAB` from the main menu. Ignore compilation problems at this step. On this step Eclipse needs some minutes to load all the files for the target platform. I'ts very important to wait with the next step until Eclipse is finished with this step.
1. Download and install [Maven 3](http://artfiles.org/apache.org/maven/maven-3/3.3.1/binaries/apache-maven-3.3.1-bin.zip). Set `MAVEN_OPTS` to `-Xms512m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m` in order to avoid memory problems during the build process.
1. Execute `mvn clean install` from the repository root. You will find the (binary) results in the folder distribution/target. On the very first code generation, you are asked on the console to download an ANTLR file, answer with "y" and press enter on the console. (See https://groups.google.com/forum/#!topic/openhab/QgABTJAkHOg if you're getting "Could not find or load main class org.eclipse.emf.mwe2.launch.runtime.Mwe2Launcher").
1. All your project in the workspace should now correctly compile without errors. If you still see error markers, try a "clean" on the concerned projects. If there are still errors, it could be that you use JDK 1.6 instead of JDK 1.7, or the JDK compliance is set to 1.4 or 1.5 instead of at least 1.6.
1. To launch openHAB from within your IDE, go to Run->Run Configurations->Eclipse Application->openHAB Runtime (resp. openHAB Designer). Switch to "Plug-in Development" perspective first (Windows->Open Perspective->Other->Plug-in Development).

After the first successful local build it might be a good idea to build with maven -o (offline) afterwards. This option accelerates the project dependency resolution by 10-20x since it lets maven search it's local repository. Normally, snapshot-enabled projects are using external repositories to find latest built packages.

Note that the IDE being installed by the above mentioned method _does **not** contain any code generation facilities_ (MWE, Xtext) itself since that setup caused difficulties on certain machines. The code generation is accomplished through the maven build though.

## 'Vagrant' Instructions

Alternatively, you may wish to use [vagrant](http://www.vagrantcloud.com) to get a pre-configured, running IDE. 

1. Install [VirtualBox](https://www.virtualbox.org/) and [vagrant](http://www.vagrantcloud.com) first.  
1. Create a new directory for vagrant and `cd` into it
1. Run `vagrant init rub-a-dub-dub/openhabdev32`
1. Execute `vagrant up` (this will take some time as a ~3GB virtual image needs to be downloaded)
1. Execute `vagrant ssh` to access your machine

To turn off the vagrant machine, run `vagrant halt` in the vagrant directory you created. Run `vagrant suspend` to just suspend the machine. To get rid of the VM and its resources, run `vagrant destroy`. For more information on the setup and use of the IDE with vagrant, look [here](https://vagrantcloud.com/rub-a-dub-dub/openhabdev32).