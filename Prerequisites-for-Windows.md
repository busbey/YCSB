YCSB is made to run on Windows systems given some additional software. The following instructions are broken into sections based on running from an existing release and building from the source. Users are encouraged to rely on published releases where possible.

# Running from a release

These instructions cover running the YCSB _client_ on your Windows machine. It does not cover any steps needed to run any datastore instances you may wish to evaluate. Please see the documentation for a given datastore to see what steps are needed to run an instance on Windows.

## Needed software

### Java Runtime Environment

The YCSB client and each datastore binding is written in the Java language. In order to run the client you will need a compatible Java Runtime Environment (JRE); YCSB is developed to work with both the Oracle Java SE and the OpenJDK runtimes. Ensure you follow the instructions to add the `java` command to your system path.

* For more information on getting Oracle Java SE JRE [see the download page for Java SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* For more information on OpenJDK [see the project website](http://openjdk.java.net/). One provider of binary installers for Windows is [Zulu](http://www.azulsystems.com/products/zulu).

After you have installed a JRE, add an environment variable named "JAVA_HOME" that points to the installation. This folder should include both folders named 'bin' and 'lib'. For example, the default location for the Oracle Java SE JRE version 1.8.0_45 is `C:\Program Files\Java\jre1.8.0_45`. For more information on setting environment variables [see the 'How to set the path and environment variables in Windows' entry on Computer Hope](http://www.computerhope.com/issues/ch000549.htm).

### Python 2.7

YCSB relies on a Python 2.7 script to handle setting up the proper runtime environment for Java. You will need to install the Python interpreter.

* For more information on downloading Python [see the Python for Windows installer page](https://www.python.org/downloads/windows/) and select the 'latest Python 2 release' 
* As of this writing, the latest Python 2 release [is Python 2.7.10 for Windows](https://www.python.org/downloads/release/python-2710/)

### Means to expand tar.gz files

YCSB uses Gzip compressed Tar files to distribute its framework, datastore bindings, and related materials. You will need some means to uncompress these files. The following are meant to be examples, if you use them you need only pick _one_:

* 7-Zip is open source software for compression available in a variety of language localizations. For more information [see the 7-Zip project page](http://www.7-zip.org/)
* Cygwin is a collection of open source tools that includes the GNU tar program. For more information [see the Cygwin project page](http://www.cygwin.com/)
* WinZip is a proprietary application for working with compressed files, with a free trail period. For more information [see the WinZip product page](http://winzip.com/windows/en/index.htm) 

For more information on opening tar.gz files see [this article from Tom Harrison](http://www.ofzenandcomputing.com/how-to-open-tar-gz-files/).

## Verify things work

1. Download the latest version of YCSB as explained in the [Getting Started guide](https://github.com/brianfrankcooper/YCSB/wiki/Getting-Started)
1. Use the program of your choice to decompress the contents of the YCSB release into a known location (e.g. `C:\YCSB-0.1.4\`)
1. Change into said directory
1. Use python to execute the `bin\ycsb` script

Example with python.exe installed in `C:\Python27\`, using the 'basic' datastore binding that only echoes transactions:
```
        C:\> cd YCSB-0.1.4
        C:\YCSB-0.1.4> C:\Python27\python.exe bin\ycsb load basic -P workloads\workloada -p recordcount=4
        â€¦.SNIPâ€¦.
```

You should see four INSERT statements, followed by some stats prefixed with [OVERALL], [CLEANUP], and [INSERT]. There should be no errors.

# Building from source

## Needed software

### Java Development Environment

In order to build the YCSB framework you will need a Java Development Environment (JDK). Please note that this is a different software distribution than the Java Runtime Environment mentioned above. YCSB should build properly with either the Oracle Java SE or OpenJDK development environments.

* For more information on getting Oracle Java SE JDK [see the download page for Java SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* For more information on OpenJDK [see the project website](http://openjdk.java.net/). One provider of binary installers for Windows is [Zulu](http://www.azulsystems.com/products/zulu).

After you have installed a JDK, add an environment variable named "JAVA_HOME" that points to the installation. This folder should include both folders named 'bin' and 'lib'. For example, the default location for the Oracle Java SE JDK version 1.8.0_45 is `C:\Program Files\Java\jdk1.8.0_45`. For more information on setting environment variables [see the 'How to set the path and environment variables in Windows' entry on Computer Hope](http://www.computerhope.com/issues/ch000549.htm).

### Maven

YCSB uses Apache Maven to manage its build process. To build from source you will need to set up Maven and ensure that it is available in your system path.

* For more information on setting up Maven see [the set of prerequisites for building with Maven on Windows](https://maven.apache.org/guides/getting-started/windows-prerequisites.html)

### Git

YCSB uses git for its version control, hosted by GitHub. The instructions below will include both downloading source directly and checking out a working copy of our version control system. In order to check out the working copy, you will need to set up access to GitHub on your machine.

* For more information on setting up git see the [help article on GitHub for Windows](https://help.github.com/articles/set-up-git/#platform-windows).

## First build

1. Obtain the source, either by using git to clone the repository as explained in the [Getting Started guide](https://github.com/brianfrankcooper/YCSB/wiki/Getting-Started) or by [downloading a zip file from the web site](https://github.com/brianfrankcooper/YCSB/archive/master.zip)
  - If you clone the git repository, change directory into the checkout before continuing
  - If you download the zip file, uncompress it into a known location (e.g. `C:\YCSB-master`) and change directory into that location before continuing
1. Run Maven with the 'package' goal to build the project and create a binary distribution tar.gz file.

Example with the downloaded source decompressed into `C:\YCSB-master\`:
```
        C:\> cd YCSB-master
        C:\YCSB-master> mvn package
        â€¦.SNIPâ€¦.
        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time: 06:14 min
        [INFO] Finished at: Mon Jul 06 00:34:07 CDT 2015
        [INFO] Final Memory: 81M/756M
        [INFO] ------------------------------------------------------------------------
```

If you so desire, you can now take the tar.gz distribution file from `distribution\target\` and use it by following the instructions above for running from a release.