This repo is a high level view of how I run operations on my home network.
It just consists of several subprojects:

* *ansible* - This is where most of the action happens.  Each role has a subproject.  (Sub-sub-project.)
* *certificate\_authority* - Tools for being a CA, operational state of my own CA, and signed certificates for my hosts.
* *os\_deployment* - Tools to apply bits to the bare metal to make the go go on the computer box.
* *git-automation* - My whacked out idea of how revision control should work.  Slow; thorough.

To get the whole collection, Do this:

    git clone --recurse-submodules git@github.com:abugher/control_center.git

To sync changes up and down, try this:

    ./git-automation/g
