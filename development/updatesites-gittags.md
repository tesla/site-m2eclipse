# Update Sites & Git Tags

### m2e download area layout

     milestones/
      1.0/
        1.0.300.<qualifier>
        1.0.300.<qualifier>
        1.0.300.<qualifier>
      1.1/
        1.1.0.<qualifier>
        1.1.0.<qualifier>
        1.1.0.<qualifier>

\
 m2e will advertise
[http://download.eclipse.org/technology/m2e/releases/](http://download.eclipse.org/technology/m2e/releases/)
as canonical m2e repository url. this repository is a composite that
will contain all released m2e version, so clients who use this url will
be able to install releases and service releases releases from the same
location. clients will be able to use release-specific repositories
(i.e.
[http://download.eclipse.org/technology/m2e/releases/1.0)](http://download.eclipse.org/technology/m2e/releases/1.0))
if they are only interested in maintenance releases.

http://download.eclipse.org/technology/m2e/milestones/ contains
milestone builds towards releases. It has similar structure as releases/
repository but without composite p2 repository at the top level. Uses
will need to pick release-specific milestone build "stream".

Composite repository descriptors are maintained by hand.

m2e does not provide snapshot/nightly builds from download.eclipse.org

### m2e git repository tags

m2e git repository tags will match download area directory structure,
i.e. "releases/1.0/1.0.0.20110607-2117",
"milestones/1.1/1.1.0.\<qualifier\>" and so on. Since builds are
promoted from milestones to releases repositories, the same build can
have multiple tags associated with it.

Tags are pushed to canonical git repository soon after build goes
"live".

### m2e release train contributions

During development, m2e will provide release train contributions via
milestone version repository (i.e.
[http://download.eclipse.org/technology/m2e/milestones/1.1).](http://download.eclipse.org/technology/m2e/milestones/1.1).)
Each new build will be automatically picked up by the release aggregator
job and does not require changes to aggregator descriptor.

During release, latest milestone build is \*copied\* to corresponding
releases/ directory and m2e contribution to release train aggregator
will need to be updated at this point.

### m2e releases

  release\   full vesion\             date\         tag\                                  p2 url\
                                                                                          

  1.0\       1.0.0.20110607-2117\     2011-06-22\   releases/1.0/1.0.0.20110607-2117\     [http://download.eclipse.org/technology/m2e/releases/1.0/1.0.0.20110607-2117](http://download.eclipse.org/technology/m2e/releases/1.0/1.0.0.20110607-2117)\
                                                                                          

  1.0 SR1\   1.0.100.20110804-1717\   2011-09-23\   releases/1.0/1.0.100.20110804-1717\   [http://download.eclipse.org/technology/m2e/releases/1.0/1.0.100.20110804-1717](http://download.eclipse.org/technology/m2e/releases/1.0/1.0.100.20110804-1717)\
                                                                                          

  1.0 SR2\   1.0.200.20111228-1245\   2012-02-24\   releases/1.0/1.0.200.20111228-1245\   [http://download.eclipse.org/technology/m2e/releases/1.0/1.0.200.20111228-1245](http://download.eclipse.org/technology/m2e/releases/1.0/1.0.200.20111228-1245)\
                                                                                          

  1.1\       1.1.0.20120530-0009\     2012-06-27\   releases/1.1/1.1.0.20120530-0009\     [http://download.eclipse.org/technology/m2e/releases/1.1/1.1.0.20120530-0009](http://download.eclipse.org/technology/m2e/releases/1.1/1.1.0.20120530-0009)\
                                                                                          
  ---------- ------------------------ ------------- ------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------
