# Development Environment

Download and unpack Eclipse latest SDK Juno milestone build from
[http://download.eclipse.org/eclipse/downloads/.](http://download.eclipse.org/eclipse/downloads/.)
Both 3.8 and 4.2 builds are expected to work.

Install latest m2e 1.1 milestone build from
[http://download.eclipse.org/technology/m2e/milestones/1.1](http://download.eclipse.org/technology/m2e/milestones/1.1)
.

Clone m2e-core repository. Committers should use
[ssh://YOUR\_COMMITTERID@git.eclipse.org/gitroot/m2e/m2e-core.git.](ssh://YOUR_COMMITTERID@git.eclipse.org/gitroot/m2e/m2e-core.git.)
For read-only access use either
[git://git.eclipse.org/gitroot/m2e/m2e-core.git](git://git.eclipse.org/gitroot/m2e/m2e-core.git)
or
[http://git.eclipse.org/gitroot/m2e/m2e-core.git.](http://git.eclipse.org/gitroot/m2e/m2e-core.git.)

Import m2e-core as existing maven project. Follow onscreen instructions,
allow m2e to install additional software and restart eclipse when
requested.

Clone m2e-core-tests repository. Developers should use
git@github.com:sonatype/m2e-core-tests.git. For read-only access use
either
[git://github.com/sonatype/m2e-core-tests.git.](git://github.com/sonatype/m2e-core-tests.git.)

Import m2e-core-tests as existing maven project.
