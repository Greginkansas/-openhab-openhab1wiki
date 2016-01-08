Follow these steps if you want to release a new version of openHAB. Later we plan to leverage the Maven Release Plugin to facilitate these steps.

1. Pull the latest state from the repository into a fresh and empty directory

        git clone git@github.com:openhab/openhab.git
1. Open a command-line and go to the `openhab` directory
1. Set project version to release version with the Tycho Versions plugin

        export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=1024m"; mvn -P prepare-release initialize -DnewVersion=1.x.0
1. Execute a Maven build

        mvn -P deploy clean deploy -Drepo.id=cloudbees-public-release-repo -Drepo.url=dav:https://repository-openhab.forge.cloudbees.com/release/1.x.0 -Dp2.repo.dir=p2 -Dapt.repo.dir=apt-repo
1. Install and extensively test the created binaries
1. Commit the changed files

        git commit -a -m "prepare for 1.x.0 release"
1. Create a release tag with pattern `v<version>`

        git tag v1.x.0
1. Increment to next development version

        mvn -P prepare-next-snapshot initialize -DnewVersion=1.y.0.qualifier
1. Execute a Maven build with goals clean verify to assure that everything builds

        mvn clean verify
1. Commit the changes

        git commit -a -m "increment to next development version 1.y.0"
1. Push the changes including the tag to the server

        git push origin master --tags
1. Switch to the root directoy of openHAB

### Manual Steps to walk through after the Release-Build

* check if start.sh and start_debug.sh has been unpacked with correct file permissions (0755)
* Issue a rough Smoke-Test (unpack runtime and demo, start runtime, access Classic-UI)
* check if Designer (for at least xxx and Windows) works
* upload deb and p2 Repository
* add new bindings to http://www.openhab.org/features-tech.html

### Upload the deb Repository to bintray
#### Fake packages
The fake packages are a workaround for triggering the metadata generation
for the arm and mips architectures (e.g. https://bintray.com/openhab/apt-repo/openhab/view#files/dists/1.7.1/main binary-armhf, binary-armel, ..)

For every new release *once-only* the fake packages have to be uploaded.
* open a command-line 
* the fake packages must be available in the current working directory
* you can download the fake packages from bintray https://bintray.com/openhab/apt-repo/fake/fake-0.1/view#files/pool/main/fake
* execute the distribution/src/deb/bintray-upload-fake-packages.sh script from your openHAB git clone
* executing without arguments will print the usage information
* all packges are automatically published
* Examples
```
sh bintray-upload-fake-packages.sh theoweiss bintray-api-key-abc gpgsecret-xyz 1.8.0.RC1
```
```
sh bintray-upload-fake-packages.sh theoweiss bintray-api-key-abc gpgsecret-xyz 1.8.0
```

#### openHAB packages
* open a web browser go to the bintray openHAB organisation, step into the apt-repo repository, add a new __Version__ to the openhab package: e.g. 1.7.0.RC2
* open a command-line and go to the local apt-repo directory
* execute the distribution/src/deb/bintray-upload-debs.sh script from your openHAB git clone
* executing without arguments will print the usage information
 * the first argument is your bintray username
 * the second argument is your bintray api key
 * the third argument is the gpg key passphrase
 * the fourth argument is the openHAB version
 * the fifth argument is the distribution name. Use:
   * "stable" for releases
   * "testing" for release canditates
   * "unstable" for snapshots
* all packges are automatically published
 * Examples
```
sh bintray-upload-debs.sh theoweiss bintray-api-key-abc gpgsecret-xyz 1.7.0-RC2 testing
```
```
sh bintray-upload-debs.sh theoweiss bintray-api-key-abc gpgsecret-xyz 1.7.0 stable
```

### Channels to inform about the new Release

1. News-Section on index.html
1. Google-Group
1. Twitter
1. Google+ Page
1. Google Community
1. KNX-User-Forum

to be continued …