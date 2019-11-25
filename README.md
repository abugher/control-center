This repo is a high level view of how I run operations on my home network.
It just consists of several subprojects:

* *ansible* - This is where most of the action happens.
* *certificate\_authority* - Tools for being a CA, operational state of my own CA, and signed certificates for my hosts.
* *os\_deployment* - Tools to apply bits to the bare metal to make the go go on the computer box.

To get the whole collection, Do this:

    git clone --recurse-submodules git@github.com:abugher/control_center.git

I have attempted to deal with the complexity of managing nested subprojects with a program named *g*, available in the *git-automation* repository:

https://github.com/abugher/git-automation
