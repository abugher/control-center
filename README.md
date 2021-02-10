This repo is how I run operations on my home network.  It consists of several
subprojects:

* *ansible* - This is where most of the action happens.  Each role has a subproject.  (Sub-sub-project.)
* *certificate\_authority* - Tools for being a CA, operational state of my own CA, and signed certificates for my hosts.
* *os\_deployment* - Tools to apply operating system images to boot media.
* *git-automation* - A tool to manage git operations.

# Collaboration

To get the whole collection, do this:

    git clone --recurse-submodules https://github.com/abugher/control_center.git

There are also several untracked subdirectories, named starting with
"sensitive\_", as reflected by *.gitignore* .  Some code may not work unless
those are populated.

If the *sensitive\_\** subdirectories are git repositories with upstream
repositories set (hopefully not public), *g* will recurse through those as
well.

If you have write access and wish to push changes, fix the remote push URL's
with this:

    ./bin/fix_remotes

To sync changes up and down, try this:

    PATH=./git-automation g

If you want to fork the whole set ... good luck.  You probably need to edit
.gitmodules, ansible/.gitmodules, and bin/fix\_remotes to reflect your own
namespace, then run ./bin/fix\_remotes .  That may still fail unless you also add
some lines to ./bin/fix\_remotes to create the repo in your namespace before
attempting a pull.  github provides a tool called "hub" which may help with
that.  ( https://github.com/github/hub )  gitlab makes that part easier,
allowing you to create a new repo by pushing to it.  (
https://docs.gitlab.com/ee/user/project/working\_with\_projects.html )

# Usage

Most of the content is in the subprojects.  There is one high level script to
generate configuration for and deploy a new host:

    ./bin/generate_host <hostname>

This will do things like assign a unique IP address, generate host
certificates, add the host to a few groups, and deploy all assigned roles to
the host.  OS installation is not included.  (See *os\_deployment* for that.)
If the host is already a member of the groups corresponding to its intended
purpose, this script should perform a complete bring-up.
