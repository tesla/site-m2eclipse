# Plugin Execution not Covered

### Background

m2eclipse 0.12 and earlier executed some parts of Maven build lifecycle
inside Eclipse and then configured the Eclipse project based on
after-execution state collected in MavenProject. This was controlled by
many different sets of maven goals -- goals when projects were imported,
project configuration changes and workspace full and incremental builds.
Some of these goals were configured at workspace level, some in
project/.settings. On top of that, there was project-level setting to
"skip" maven-compiler-plugin execution.

Unfortunately, this did not work well or not at all for many projects.
Probably even worse, it did not \*always\* work for many projects, so we
had to go through series of refresh/update dependencies/update
configuration/rebuild voodoo (or "m2eclipse dance" as some called it) to
get projects in a good state. For example
[MNGECLIPSE-823](https://issues.sonatype.org/browse/MNGECLIPSE-823) was
the most voted issue in m2e jira and it was a direct manifestation of
this "flakiness".

Most, if not all, such problems were traced back to one of two root
causes. 1. Out-of-workspace resource changes made by Maven plugin
triggered unexpected workspace builds. This was very indeterministic. In
some cases projects appeared to work fine. In some cases,
generated/filtered resources would go missing. And in some cases
workspace build would go on forever. 2. Various JVM and OS resources
leaks by Maven plugins was another common cause of problems.

To solve these long-standing issues, m2e 1.0 requires explicit
instructions what to do with all Maven plugins bound to "interesting"
phases (see [M2E interesting lifecycle
phases](M2E%20interesting%20lifecycle%20phases)) of a project build
lifecycle. We call these instructions "project build lifecycle mapping"
or simply "lifecycle mapping" because they define how m2e maps
information from project pom.xml file to Eclipse workspace project
configuration and behaviour during Eclipse workspace build.

Project build lifecycle mapping can be configured in a project's
pom.xml, contributed by Eclipse plugins, or defaulted to the commonly
used Maven plugins shipped with m2e. We call these "lifecycle mapping
metadata sources". m2e will create error marker like below for all
plugin executions that do not have lifecycle mapping in any of the
mapping metadata sources.

    Plugin execution not covered by lifecycle configuration:
    org.apache.maven.plugins:maven-antrun-plugin:1.3:run
        (execution: generate-sources-input, phase: generate-sources)

m2e matches plugin executions to actions using combination of plugin
groupId, artifactId, version range and goal. There are three basic
actions that m2e can be instructed to do with a plugin execution --
**ignore**, **execute** and **delegate** to a project configurator.

### delegate to a project configurator (recommended)

**configurator** mapping tells m2e to delegate workspace project
configuration mapping for matching plugin execution to an implementation
of AbstractProjectConfigurator registered with m2e using
projectConfigurators extension point.

In most cases configurator mapping will be used by m2e extension
developers. See [M2E Extension
Development](M2E%20Extension%20Development) wiki page for more
information about developing m2e extensions.

### ignore plugin goal

**ignore**, as the name suggests, tells m2e to silently ignore the
plugin execution. Here is sample pom.xml snippet

    <pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.eclipse.m2e</groupId>
          <artifactId>lifecycle-mapping</artifactId>
          <version>1.0.0</version>
          <configuration>
            <lifecycleMappingMetadata>
              <pluginExecutions>
                <pluginExecution>
                  <pluginExecutionFilter>
                    <groupId>some-group-id</groupId>
                    <artifactId>some-artifact-id</artifactId>
                    <versionRange>[1.0.0,)</versionRange>
                    <goals>
                      <goal>some-goal</goal>
                    </goals>
                  </pluginExecutionFilter>
                  <action>
                    <ignore />
                  </action>
                </pluginExecution>
              </pluginExecutions>
            </lifecycleMappingMetadata>
          </configuration>
        </plugin>
      </plugins>
    </pluginManagement>

HINT: m2e provides a quick-fix associated with "plugin execution not
covered" to easily create \<pluginManagement/\> elements like above.

### execute plugin goal

**execute** tells m2e to execute the action as part of Eclipse workspace
full or incremental build. Beware that m2e does not provide any
safeguards against rogue maven plugins that leak classloaders, modify
random files inside workspace or throw nasty exceptions to fail the
build. Use this as the last resort and make sure you know what you are
doing.

<pluginManagement>
  <plugins>
    <plugin>
      <groupId>org.eclipse.m2e</groupId>
      <artifactId>lifecycle-mapping</artifactId>
      <version>1.0.0</version>
      <configuration>
        <lifecycleMappingMetadata>
          <pluginExecutions>
            <pluginExecution>
              <pluginExecutionFilter>
                <groupId>some-group-id</groupId>
                <artifactId>some-artifact-id</artifactId>
                <versionRange>[1.0.0,)</versionRange>
                <goals>
                  <goal>some-goal</goal>
                </goals>
              </pluginExecutionFilter>
              <action>
                <execute >
                  <runOnIncremental>false</runOnIncremental>
                </execute >
              </action>
            </pluginExecution>
          </pluginExecutions>
        </lifecycleMappingMetadata>
      </configuration>
    </plugin>
  </plugins>
</pluginManagement>

 HINT: use quick fix to create "ignore" mapping, then replace
\<ignore/\> action with \<execute/\>

### metadata source lookup order

m2e considers lifecycle mapping metadata sources in the following order

1.  pom.xml file of the project
2.  parent, grand-parent and so on pom.xml files
3.  installed m2e extensions (in no particular order)
4.  [m2e] lifecycle mapping metadata provided by maven plugin (see
    below)
5.  default lifecycle mapping metadata shipped with m2e

m2e uses the first applicable mapping found.

### lifecycle mapping metadata provided by maven plugin

Starting with m2e 1.1, maven plugin developers are able to provide
lifecycle mapping metadata as part of the plugin itself. If present,
such mapping metadata will be automatically used by m2e, thus
eliminating the need for plugin specific project configurator and/or
lifecycle mapping metadata in pom.xml.

[M2E compatible maven plugins](M2E%20compatible%20maven%20plugins) wiki
page provides more information about developing m2e-compatible maven
plugins that do not require external build lifecycle mapping
configuration.

## m2e maven plugin coverage status

<table>
  <tr>
    <th>groupId</th>
    <th>artifactId</th>
    <th>version range</th>
    <th>goals</th>
    <th>status</th>
  </tr>
  <tr>
    <td>org.antlr</td>
    <td>
      <a href="http://www.antlr.org/antlr3-maven-plugin/">antlr3-maven-plugin</a>
    </td>
    <td>[3.1.1,)]</td>
    <td>antlr</td>
    <td>
      <a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.antlr">
        org.sonatype.m2e.antlr
        <a />
    </td>
  </tr>
  <tr>
    <td>org.apache.maven.plugins</td>
    <td><a href="http://maven.apache.org/plugins/maven-jar-plugin/">maven-jar-plugin</a></td>
    <td>[2.0)]</td>
    <td>jar</td>
    <td>
      <a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.mavenarchiver.pomproperties">org.sonatype.m2e.mavenarchiver.pomProperties</a></td>
  </tr>
  <tr>
    <td>org.codehaus.modello</td>
    <td><a href="http://modello.codehaus.org/modello-maven-plugin/">modello-maven-plugin</a></td>
    <td>[1.0.1,)]</td>
    <td>java</td>
    <td><a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.modello">org.sonatype.m2e.mavenarchiver.pomProperties</a></td>
    <td>xpp3-reader, xpp3-writer, xpp3-extended-reader,xsd,stax-reader</td>
  </tr>
  <tr>
    <td>org.codehaus.mojo</td>
    <td><a href="http://mojo.codehaus.org/antlr-maven-plugin/">antlr-maven-plugin</a></td>
    <td>[2.1,)</td>
    <td>generate</td>
    <td><a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.antlr">org.sonatype.m2e.antlr</a></td>
  </tr>
  <tr>
    <td>org.codehaus.mojo</td>
    <td><a href="http://mojo.codehaus.org/build-helper-maven-plugin/">build-helper-maven-plugin</a></td>
    <td>[1.0,)</td>
    <td>add-source, add-test-source</td>
    <td><a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.buildhelper">org.sonatype.m2e.buildhelper</a></td>
  </tr>
  <tr>
    <td>org.codehaus.plexus</td>
    <td><a href="http://plexus.codehaus.org/plexus-containers/plexus-component-metadata/">plexus-component-metadata</a></td>
    <td>[1.0-beta-3.0.6,)</td>
    <td>generate-metadata, generate-test-metadata</td>
    <td><a href="https://github.com/sonatype/m2eclipse-extras/tree/master/org.sonatype.m2e.plexus.annotations">org.sonatype.m2e.plexus.annotations</a></td>
  </tr>
  <tr>
    <td>org.codehaus.plexus</td>
    <td><a href="http://plexus.codehaus.org/plexus-maven-plugin/">plexus-maven-plugin</a></td>
    <td>[1.1,)</td>
    <td>descriptor, test-descriptor, merge-descriptors, test-merge-descriptors</td>
    <td>not supported</td>
  </tr>

</table>

## Help improve m2e maven plugin coverage

First and foremost, you need to understand the desired behaviour. In
most cases this should be limited to IDE usecase, i.e. editing sources
and running tests, and not the complete Maven build, so plugin goals
that publish build results to a remote repository can be ignored without
any adverse side effects, while java source code generation most likely
is necessary.

If the desired behaviour is applicable to other Maven projects using the
plugin goal, we strongly recommend documenting your findings in m2e
[bugzilla](https://bugs.eclipse.org/bugs/). Please use "[mojo]
plugin-artifact-id:goal support" bugzilla summary and make sure to
[search for existing
records](https://bugs.eclipse.org/bugs/buglist.cgi?query_format=specific&amp;amp;amp;amp;amp;amp;amp;order=relevance+desc&amp;amp;amp;amp;amp;amp;amp;bug_status=&amp;amp;amp;amp;amp;amp;amp;product=m2e&amp;amp;amp;amp;amp;amp;amp;content=mojo).
When submitting new request, please provide standalone example project
and detailed description of desired behaviour when the project is
imported in Eclipse workspace. This will allow other users and
interested developers to track popularity of various Maven plugins and
schedule implementation work accordingly.

[M2E Extension Development](M2E%20Extension%20Development) has pointers
how to develop m2e extensions.

Common problems
-----------------------------------------------------------------------------

Some Maven plugins are recognized as problematic and will produce error
markers with a text similar to: *maven-dependency-plugin (goals
"copy-dependencies","unpack") is not supported by m2e*

In version 1.0 there is no quick fix available for this but it is
possible to define a lifecycle mapping for the plugin as well (as shown
in *ignore plugin goal* above). Which removes the error marker.
