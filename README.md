# Introduction

This repo is how I run operations on my home network.  It consists of several
subprojects:

* *ansible* - This is where most of the action happens.  Each role has a subproject.  (Sub-sub-project.)
* *certificate-authority* - Tools for being a CA, operational state of my own CA, and signed certificates for my hosts.
* *os-deployment* - Tools to apply operating system images to boot media.
* *git-automation* - A tool to manage git operations.


# Collaboration

## Partial

You should really be able to just get one or a very small number of core repos,
like `control-center` and/or `ansible`, along with the exact roles you want and
those roles they depend on.  You can probably do so now, if you use `git`
intelligently and don't try to use `g`.  I suspect that my submodule
arrangement makes this awkward, though.  See BUGS.

For now, refer to the `Full` instructions, below.  If you clone the repo
without recursing submodules, then edit and trim down the submodule list, you
may then be able to recursively sync the submodules and get only the set of
roles you want.  **Untested.**

## Full

To get the whole collection, do this:

    git clone --recurse-submodules https://github.com/abugher/control-center.git

There are also several untracked subdirectories, named starting with
"sensitive-", as reflected by *.gitignore* .  Some code may not work unless
those are populated.

If you have write access and wish to push changes, fix the remote push URL's
with this:

    ./bin/fix-remotes

To sync changes up and down, try this:

    PATH=./git-automation g

If you want to fork the whole set ... good luck.  You probably need to edit
.gitmodules, ansible/.gitmodules, and bin/fix\_remotes to reflect your own
namespace, then run ./bin/fix\_remotes .  That may still fail unless you also add
some lines to ./bin/fix\_remotes to create the repo in your namespace before
attempting a pull.  github provides a tool called "hub" which may help with
that.  ( https://github.com/github/hub )  gitlab makes that part easier,
allowing you to create a new repo by pushing to it.  (
https://docs.gitlab.com/ee/user/project/working_with_projects.html )


# Usage

Most of the content is in the subprojects.  There is one high level script to
generate configuration for and deploy a new host:

    ./bin/generate-host <hostname>

This will do things like assign a unique IP address, generate host
certificates, add the host to a few groups, and deploy all assigned roles to
the host.  OS installation is not included.  (See *os\_deployment* for that.)
If the host is already a member of the groups corresponding to its intended
purpose, this script should perform a complete bring-up.


# BUGS

Submodules relationships may need to be inverted.  Presently, the big picture
contains all the smaller pictures.  Rather than contain, each smaller picture
should depend on a larger picture.  For example, if an ansible role repo has
the central ansible repo as a submodule, then recursively syncing the ansible
role repo gets the central ansible components installed.  On the other hand,
that might imply a separate copy of the bigger picture repo in each smaller
picture repo.  Either approach seems wrong.
