# Marketplace Entries

## Submitting new m2e marketplace entries

1.  Open a bugreport in [m2e
    bugzilla](https://bugs.eclipse.org/bugs/enter_bug.cgi?product=m2e)
    with "[catalog] your extension name" in the subject.
2.  In the bugreport please provide the following information about your
    m2e extension. Use [this XML descriptor
    format](http://git.eclipse.org/c/m2e/m2e-discovery-catalog.git/tree/org.eclipse.m2e.discovery.oss/connectors.xml),
    in other words, we'd like to be able to paste provided entry
    information directly to the descriptor. For updates to existing
    entries, please provide diffs we can apply with patch or git.
    -   name, description and summary (don't ask what's the difference
        between the last two)
    -   license and provider information
    -   p2 repository and installable unit information of the extension
        (see below)
    -   sample project and steps to see your extension in action
    -   optionally, url of the extension source repository.

Existing submissions are listed in [this XML
descriptor](http://git.eclipse.org/c/m2e/m2e-discovery-catalog.git/tree/org.eclipse.m2e.discovery.oss/connectors.xml)


## Updating existing marketplace entries

1.  Clone git.eclipse.org/c/m2e/m2e-discovery-catalog.git
2.  Make desired changes to org.eclipse.m2e.discovery.oss/connectors.xml
3.  Verify your changes by running "mvn clean install" from the root of
    the repository
4.  Commit your changes
5.  Generate patch using "git format-patch HEAD\^" command
6.  Open new bugreport in m2e bugzilla with "[catalog] your extension
    name" in the subject and attach the patch to the bugzilla

## Maven repositories

Due to [bug
346830](https://bugs.eclipse.org/bugs/show_bug.cgi?id=346830), support
for resolving m2e marketplace catalog entries from Maven repositories
has been disabled.

## P2 repositories

For m2e extensions published in p2 repositories we need information
about p2 repository url, installable unit id of the feature or bundle to
be installed and installable unit id of a bundle that contains
lifecycle-mapping-metadata.xml

Make sure extension's installable unit contains applicable license
information. In practical terms, this means extension must have
feature.xml files with \<license\>\</license\> element.

It is assumed that contents of provided p2 repository will not change in
the future unless corresponding m2e marketplace catalog entry is removed
or amended.
