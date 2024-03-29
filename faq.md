# FAQ

## Common problems

### Unable to download the artifact from any repository xxx:zzz-2.4.1.jar

In some project dependency resolved and Maven builder give weird error
about magic dependency version 2.4.1. This happens when project poms are
using `${version}`, which is resolved to system property that is set by
some other Eclipse plugin. See
[MGG-2653](http://jira.codehaus.org/browse/MNG-2653) for more details.

### Unable to locate the Javac Compiler Error

Some users reported the following error that happens when importing
Maven projects or when invoking "Update Sources" action.

    6/25/07 1:15:44 PM CDT: ERROR mojo-execute : compiler:compile : Compilation failure
    Unable to locate the Javac Compiler in:
    C:Program FilesJavaj2re1.4.2_14..libtools.jar
    Please ensure you are using JDK 1.4 or above and
    not a JRE (the com.sun.tools.javac.Main class is required).
    In most cases you can change the location of your Java
    installation by setting the JAVA_HOME environment variable.

That happens because those actions runs in the same JVM where Eclipse is
running. If that JVM comes from JRE that isnÃ¢Â™t part of JDK, there is
no Java compiler (the tools.jar) around. To workaround this you can add
`-vm` argument to Eclipse command line or eclipse.ini. For Eclipse 3.3
it would look something like this:

    -showsplash
    org.eclipse.platform
    --launcher.XXMaxPermSize
    256m
    -vm
    C:\jdk1.6.0\bin\javaw.exe
    -vmargs
    -Xms40m
    -Xmx256m

Alternatively you could specify compilerId in the pom.xml, so Maven
wonÃ¢Â™t be looking for JDK when compiling Java code:

    <plugin>
      <artifactid>maven-compiler-plugin</artifactid>
      <configuration>
        <compilerid>eclipse</compilerid>
      </configuration>
      <dependencies>
        <dependency>
          <groupid>org.codehaus.plexus</groupid>
          <artifactid>plexus-compiler-eclipse</artifactid>
          <version>xxxx</version>
        </dependency>
      </dependencies>
    </plugin>

Also note that to launch Maven from within Eclipse, JRE used for launch
also need to come from JDK. By default Eclipse registers JRE it is
started in, but this can be configured on "Window / PreferencesÃ¢Â¦ /
Java / Installed JREs" preference page.

### Compilation errors on restricted classes

Projects using classes from `rt.jar`, such as `com.sun.*` (and some
others) can have compilation errors like: "Access restriction: The type
RE is not accessible due to restriction on required library
\<jrepath\>\</jrepath\>/lib/rt.jar". Such errors indicate use of non-API
classes and those access rules are defined by Eclipse JDT.

You can change compiler settings to not fail on those restrictions in
workspace settings in Window / Preferences / Java / Compiler /
Errors/Warnings / Deprecated and restricted API / Forbidden reference
(access rules) / Warnings; or per-project from Project / Properties /
Java Compiler / Errors/Warnings / Deprecated and restricted API /
Forbidden reference (access rules) / Warnings

### How to Configure Proxy and location of Maven local repository

Eclipse Plugin is using MavenÃ¢Â™s
[http://maven.apache.org/settings.html|`settings.xml`](http://maven.apache.org/settings.html|&lt;tt&gt;settings.xml&lt;/tt&gt;)
for proxy, local repository location and any other environment-specific
configuration. This way we can use same settings between the command
line and the IDE.

Default location of the `settings.xml` is at
\<user\>/.m2/settings.xml&lt;/tt&gt;\</user\>, but you can also specify
location of the global settings, i.e. one in
\<maven\>/conf/settings.xml&lt;/tt&gt;\</maven\>

### Classpath container refresh job never finishes

This could happens for a number of reasons, but to narrow it down you
can open Maven console in Eclipse Console view and check if there is any
logging going on. It is possible that dependency resolved got into a
loop (i.e.
[http://jira.codehaus.org/browse/MNGECLIPSE-348|MNGECLIPSE-348](http://jira.codehaus.org/browse/MNGECLIPSE-348|MNGECLIPSE-348).
As a workaround you can terminate this job and try to run Project /
CleanÃ¢Â¦ on individual projects.

If there is no logging happens then it is possible that job is blocked
by some other plugins. To check that you need to take a thread dump and
see if there are any deadlocks.

### Why resource folders in Java project have excluded="\*"

Many users are confused that when Java project is configured in Eclipse,
resource folders gets `excluded="*"`. This is done intentionally,
because those resources are processed by the "Maven Builder" registered
to the project. This builder provides special resource handling, that
includes filtering, as well as execution of other Maven plugins for
processing resources. See Maven build lifecycle for more details.

If you manually remove resource exclusion, JDT will copy resources and
overwrite filtered resources generated by Maven plugins.

Actually project resource folder doesnÃ¢Â™t really need to be added to
the buildpath (Maven Builder is going to work without it), but it been
considered convenient and look better in the Package Explorer and other
Eclipse views.

Also note, that classpath used for JUnit and Java Application launch
configurations for projects that have Maven support enabled is also
calculated in a special way and excluded resources does not affect it
either.

### Why generated source folders are not added to classpath

Maven plugins used to generate source code from resources or other
sources can register additional source folders to Maven project during
the build. Usually such plugins are bound to `process-resources` (or
`process-test-resources`) build phase (for example jaxb, modello or
xdoclet plugins). This means that to get those source folders for
generated sources, we have to run corresponding Maven build phase.

Not all projects using generated sources, so for performance reasons,
m2eclipse does not run any Maven goals by default on project import.
This can be changed in the Maven settings in "Window / PreferencesÃ¢Â¦ /
Maven / Goals to run on project import" (e.g. you can specify
"process-resources" build phase or specific plugins in that field).

Alternatively you can run "Maven / Update project configuration" action
from the project popup menu, which is configured to run
"process-resources" by default and it can be also changed on the same
preference page.

### How to configure Maven project to use separate output folders in Eclipse

Because Eclipse JDT is managing all changes in the project sources and
incrementally compiles classes it is quite sensitive to the external
modifications of compiled classes. That is why Maven users who also want
to work with their projects in Eclipse IDE also want to configure
Eclipse to use separate output folders for Maven projects, so it
wonÃ¢Â™t interfere with Maven build run from the command line.

On the other hand, many Maven plugins have assumptions about location of
compiled classes and resources and in many cases, changing output
folders would simply break those Maven plugins. That is why we decided
to remove this option from version 0.9.4, which caused inconvenience for
several people.

While we are still looking for a better solution, there is a simple
workaround that allows to configure Maven project to use separate output
folders for Eclipse. The idea is to use property to specify output
folder location and change that property using Maven profiles. Here is
how it could look like in pom.xml:

    <project>
      ...
      <build>
        <outputdirectory>${basedir}/${target.dir}/classes</outputdirectory>
        <testoutputdirectory>${basedir}/${target.dir}/test-classes</testoutputdirectory>
      </build>
     
      <properties>
        <target.dir>target</target.dir>
      </properties>
     
      <profiles>
        <profile>
          <id>eclipse-folders</id>
          <properties>
            <target.dir>target-eclipse</target.dir>
          </properties>
        </profile>
      </profiles>
      ...
    </project>      

So, by default Maven would compile classes to /target/classes folder,
but if "eclipse-folders" profile is enabled it would instead use
/target-eclipse/classes folder. This configuration can be also declared
in a common parent pom.xml.

In m2eclipse you can specify active profile for given project on Maven
property page in project properties dialog.

TODO add image

### M2HOME environment variable

Maven Integration for Eclipse currently is not using M2HOME environment
variable and generally any other environment variables.

### Why projects are renamed on import

When projects are imported directly into Eclipse workspace folder, the
project name should match folder name. It is not an issue if you specify
an alternative location for those projects.

Also, by default, Maven project import/checkout is using project name
template like [artifactId]. You can specify alternative name template in
Advanced section of the project import wizards (e.g. you can use
[artifactId]-[version] template). Note that you need to make sure that
artifact versions are different between those projects, or else
dependency resolver will get really confused.

### Why does m2eclipse checkout Maven projects as "maven.NNNNNNNN"?

The Maven project checkout is based on generic Maven SCM API (which
supports number of SCM providers out of the box) and we donÃ¢Â™t have
any information about Maven projects before the actual checkout (e.g. we
donÃ¢Â™t know if folder or project names are going to be conflicting).
There are two scenarios:

1.  the root checked out folder has pom.xml
2.  the checked out folder has number of projects without pom.xml at the
    root

The second scenario could happen when there is several projects located
at selected SCM URL or when user selected multiple SCM URLs, e.g. using
one of the available SCM UI integrations (CVS, Subclipse, etc).

In the first case the checkout folder will be renamed to match the
result Eclipse project name for the Maven pom.xml and in the first case
folder stays with "maven.NNNNNNNN" name. It is not clear if it is always
possible to move checkout folder around for all SCMs (e.g. because of
the SCM metadata).

Also see [Why projects are renamed on
import](#Why%20projects%20are%20renamed%20on%20import)

### How to connect to https repositories

The Maven howto describes how to configure Maven to work with https
repositories. To make the same properties work in Eclipse they need to
be specified in two additional places:

1.  to make in-process Maven use them (e.g. on project import or "Maven
    / Update project configuration" actions) add them to -vmargs section
    in eclipse.ini or to Eclipse command line
2.  to make Maven launch configuration use them use "Window /
    Preferences / Java / Installed JREs / Edit / Default VM arguments"

To make both Maven cli and Eclipse use the preferences they would need
to come from the MavenÃ¢Â™s settings.xml, but that would require changes
in the Maven itself.

## Miscellaneous

### How Search Works

Dependency search is using local index for Maven repositories:

-   indexes for remote Maven repositories, such as Central repository,
    can be downloaded from remote repositories if they publish index
    created using Nexus Indexer tool or if repository is managed by
    Nexus repository manager
-   indexes for remote Maven repositories can be also packaged as
    Eclipse plugin and installed using Eclipse Update manager.
-   index for a Local Maven repository is updated incrementally when
    plugin downloads jars from any remote repositories. Local repository
    can be also reindexed from Window / PreferencesÃ¢Â¦ / Maven
    preferences page.
-   index for an Eclipse workspace is updated when projects are added,
    removed or changed in Eclipse workspace.

### Maven Integration for Eclipse vs. Maven eclipse:eclipse plugin

The Maven Integration for Eclipse (m2eclipse) is an Eclipse plugin that
allows execution of Maven goals and manages Maven dependencies. It is a
different beast to the maven-eclipse-plugin which is a Maven plugin that
attempts to manage/modify Eclipse project files to account for Maven
dependencies. Generally, if you are using m2eclipse you donÃ¢Â™t really
need maven-eclipse-plugin. The former is providing advanced project
import and configuration features and provides integration with other
Eclipse tools.

As of maven-eclipse-plugin-2.3 its dependency management is incompatible
with m2eclipse and is waiting for the patches to be applied. See
MECLIPSE-78.

### What Maven version is used by plugin

Plugin is not actually using Maven itself. It is using component that is
part of Maven called Maven Embedder. This component is not available for
Maven 2.0.x. The Embedder is used by the Maven command line interface
(CLI) starting from version 2.1 that includes number of improvements to
allow it to actually embed Maven.

The m2eclipse is currently using the Embedder component from Maven 3.0.
If you want to execute particular version of Maven installed elsewhere,
you can do so from the Maven launch configuration or select it as
default in Maven / Installations preference page.

### Why is it working from the command line but not in m2eclipse

Because m2eclipse is using the embedded Maven runtime in Maven 3.0, you
can see differences in the execution between m2eclipse and command line
when Maven 2.1.x is used. We hope that such regressions and
incompatibilities will be fixed in the future releases of Maven 3.0.
Note that m2eclipse is always using embedded Maven runtime when running
Maven builder, importing projects and updating project configuration,
however you can configure m2eclipse to use external Maven installation
when launching Maven using "Run asÃ¢Â¦ / Maven XXX" actions.

### How to generate a thread dump

-   On Windows press Ctrl-Break in the Java console started with Eclipse
    IDE. To start Eclipse with Java console use java.exe instead of
    javaw.exe in eclipse.ini.

        -vm
        C:\jdk1.6.0_03\bin\java.exe
    
-   On Unix, Linux, Mac OS X press `Ctrl-\` in the terminal console used
    to start Eclipse IDE
-   Alternatively on Unix, Linux, Mac OS X send the QUIT signal to the
    Java VM running Eclipse: kill -QUIT processid, where processid is
    the process number of the respective java process
-   All OS: use jps and jstack tool from JDK 1.5 or JDK 6.0 installation
    (run jps to get the process id, jstack \<pid\>\</pid\> to print
    thread dump).
-   All OS: use StackTrace tool
    -   launch from
        [http://www.adaptj.com/root/webstart/stacktrace/app/launch.jnlp](http://www.adaptj.com/root/webstart/stacktrace/app/launch.jnlp)
    -   details
        [http://www.adaptj.com/root/main/stacktrace](http://www.adaptj.com/root/main/stacktrace)
    -   to generate thread dump invoke "Process / SelectÃ¢Â¦" from the
        main main menu and select Java process from the drop down list,
        then invoke "Process / Thread Dump"

-   Other tools listed on Javapedia ThreadDump page

### How to retrieve an actual command line used to start JVM and Maven

An actual command line used to start JVM processes in Eclipse, including
Maven builds, can be retrieved from the Debug view (e.g. from the Debug
perspective):

debugview.png

There you can select corresponding Java process and open its properties
(from the popup menu or using Alt-Enter shortcut):

processproperties.png
