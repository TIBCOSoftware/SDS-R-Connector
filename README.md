
R Connector for TIBCO Spotfire® Data Science
==================

This project provides the R server connector for Spotfire® Data Science.

Quick Start
-----------

To get the project running:

(1) Create a new build by executing: ./sbt assembly
(2) cd to ./scripts
(3) make a symlink to the assembled R Connector for Spotfire Data Science: ln -s ../server/target/scala-2.10/alpine-r-connector.jar
(4) start the server by executing: ./start_services.sh

Licensing
---------

For licensing reasons, the master project was broken up into subprojects, each of which may have independent licensing. Specifically:

- The server depends on the [Rserve/RserveEngine jar files](http://rforge.net/Rserve/files/), which are [licensed](http://rforge.net/Rserve/) under [GPL](http://www.gnu.org/copyleft/gpl.html) (probably v3, but not specified by the project's author). The server also depends on [Akka](http://akka.io/), which is licensed under ASF 2.0. With GPL being the more commerically restrictive license, and with the Apache-licensed software [being OK to include in GPL-licensed projects](http://www.apache.org/licenses/GPL-compatibility.html), the server is licensed as GPL v3. The source files under GPL include the GPL text in the header. The license itself is provided under the server project's [root directory](https://github.com/TIBCOSoftware/SDS-R-Connector/tree/master/server).
- The Akka message case classes [messages/ subproject](https://github.com/TIBCOSoftware/SDS-R-Connector/tree/master/messages) have no dependencies at all, they are licensed under [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0.html).
- The client is provided for demonstration purposes only - it does not serve any particular purpose. Since it depends on Akka and the previously mentioned message case classes, it is licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0.html).
- The [master build file](https://github.com/TIBCOSoftware/SDS-R-Connector/blob/master/project/Build.scala) and related files in the [project/ directory](https://github.com/TIBCOSoftware/SDS-R-Connector/tree/master/project) do not depend on the software it is meant to build. They can be used to build any Java/Scala code, therefore they are not bound by any license. However, since the build file currently lists the Akka dependencies in the build definition (but not the GPL Rserve jars, which are [unmanaged](http://www.scala-sbt.org/0.13.2/docs/Getting-Started/Library-Dependencies.html)), the build files are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0.html). 


Sample Build and Execution
--------------------------

You do not need to use the master build file if you do not want to - particularly if you don't care about the sample client demo. However, for a quick demo, do the following:

1. Install R. This can be done in many ways, e.g.: 
  - CentOS/RHEL:
      - install EPEL, e.g.
      
        ```sh
          $rpm -ivh http://mirror.chpc.utah.edu/pub/epel/6Server/x86_64/2ping-2.0-2.el6.noarch.rpm
        ```
      - install R
      
        ```sh
          $yum install R
        ```
  - Fedora:
      - install R
      
        ```sh
          $yum install R
        ```
  - Ubuntu:
      - install R
      
        ```sh
          $sudo apt-get install R
        ```
  - In general, you can simply wget the existing RHEL/Debian and other packages from [here](http://cran.r-project.org/bin/linux/)
  - For other options, see [here](http://cran.r-project.org/mirrors.html).
2. Run the R script found [here](https://github.com/TIBCOSoftware/SDS-R-Connector/blob/master/server/scripts/RunRserve.R). It will install the Rserve package from [CRAN]() if it's not already installed, start the Rserve TCP listener, and keep R running until the master R process terminates. For extra protection, use [nohup](http://en.wikipedia.org/wiki/Nohup) as follows:

   ```sh
     $nohup R CMD BATCH server/scripts/RunRserve.R ./r_log.txt &
   ```
   
3. Check the log file ("r_log.txt" in the above example for failures).
4. Get SBT 0.13.2 or later (warning, the build was only tested with this specific version). You can find all the information for your OS [here](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html).
5. Start SBT at the root of the project, i.e. [here](https://github.com/TIBCOSoftware/SDS-R-Connector). Use the interactive mode the first time to get feedback about each step, instead of having SBT fail one task and shutting down.

   ```sh
    $sbt
       // in case you need to reload the build definition
     > reload 
       // update dependencies
     > update
       // clean target directory
     > clean 
       // compile all projects
     > compile 
       // run main class in the sample_client subproject
     > sample_client/run 
      // run unit tests (currently only for the server)
     > test
       // package jars without dependencies (will need to be on the classpath)
     > package
      // assemble "uber jar" file with all dependencies
      // (Scala, Akka, Rserve, message beans, etc.)
     > assembly
      // assemble just the server (with Scala/Akka/Rserve/messages)
      // but without the sample_client code
     > server/assembly
    ```
After running package/assembly, pick up the jars from their respective directories. For example, the messages jar shouldn't contain dependencies unless the client code doesn't have scala-library.jar on the path. If it does, then you can build the messages jar for your own client code as follows on the sbt shell:
  ```sh
  $sbt messages/package
  ```
and then you can pick up the jar from messages/target/scala-2.10/messages_2.10-0.1.jar. If your client does not have scala-library.jar on its claspath, you can make an assembly file instead
  ```sh
  $sbt messages/assembly
  ```
and you can pick up the jar from messages/target/scala-2.10/messages-assembly-0.1.jar.
The server should be amost surely built using assembly as opposed to package, so you can do
  ```sh
  $sbt server/assembly
  ```
and you can then pick up the jar from server/target/scala-2.10/server-assembly-0.1.jar.
6. The Typesafe config file found in the config subdirectory of this project can be used as a set of defaults for the server. Once you have the server assembly jar and the config file, copy them to any directory you wish. Here I assume that the jar and the config are in the same directory, but they can be different if the config's path is correctly specified as either relative or absolute. For example, for the jar and the config in the same directory, you can start the server as follows:

  ```sh
    $java -Dconfig.file=./application.conf -jar ./server-assembly-0.1.jar
  ```
If the conf file is in a different directory, simply specify the desired path.
7. Note that you can change certain parameters of the Akka R server in the config file. For example, you can choose the server to run on a port different than 2553, and the client to run on a port different than 2552. In fact, you could remove the client section altogether. The minimal server configuration serves as the default.
8. For your own client code builds, you will need the message jar and the Akka library. The code found in the client/ subproject shows an example. You can publish the messages jar in your local SBT/ivy2 repository using sbt

  ```sh
  $sbt messages/publish-local
  ```

or you can publish it in your favorite repository management system. You will also need to add the correct Akka and Scala library versions to your client project. Currently, the project uses Scala 2.11.8 and Akka 2.3.11.

Startup/shutdown
----------------

After ensuring that R is installed, execute the [prepare_services.sh](https://github.com/TIBCOSoftware/SDS-R-Connector/blob/master/scripts/prepare_services.sh) script within the scripts/ directory to install essential packages for the R server.

The processes can be started using the [start_services.sh](https://github.com/TIBCOSoftware/SDS-R-Connector/blob/master/scripts/start_services.sh) and [stop_services.sh](https://github.com/TIBCOSoftware/SDS-R-Connector/blob/master/scripts/stop_services.sh) shell scripts. Also see [this page](https://alpine.atlassian.net/wiki/display/V6/R+Execute) .


Final Notes
-----------

The R Connector for Spotfire Data Science was shown to work on CentOS 5 and 6, with Oracle and OpenJDK 6 , SBT 0.13.2, Scala 2.11.8, Akka 2.3.11 and R 3.0.3 ("Warm Puppy"). 
