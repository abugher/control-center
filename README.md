This repo is how I run operations on my home network.  It consists of several
subprojects:

* *ansible* - This is where most of the action happens.  Each role has a subproject.  (Sub-sub-project.)
* *certificate\_authority* - Tools for being a CA, operational state of my own CA, and signed certificates for my hosts.
* *os\_deployment* - Tools to apply operating system images to boot media.
* *git-automation* - A tool to manage git operations.

To get the whole collection, do this:

    git clone --recurse-submodules git@github.com:abugher/control_center.git

There are also several untracked subdirectories, named starting with
"sensitive\_", as reflected by *.gitignore* .  Some code may not work unless
those are populated.

To sync changes up and down, try this:

    PATH=./git-automation g
