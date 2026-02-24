# Terminology

## "subproject" vs "submodule"

`man git submodule` refers to repos introduced as the subordinate end of a submodule relationship as `subprojects`.  I do the same.  Some of this might make more sense to you with the word `submodule` where you read `subproject`, depending on how you're used to discussing these things.

# Structure

## Operational Structure

These directories are used in deploying and managing code.  Most are subprojects.  See [Tools](#tools) for further details.

* `ansible-roles` - Full collection of my ansible roles.  If you like one, clone it recursively.  Expect to have to replace some subprojects, like inventory, with your own.
* `dicelessware` - Password generation.
* `git-automation` - Provides a tool, `g`, used to manage git operations on nested repos.
* `os-deployment` - Tools to apply operating system images to boot media.
* `bin` - Not a subproject.  Scripts for managing and using the contents of `control-center` (this repo).
* `cache` - Not a subproject.  Ignored by git.  Repos in here are for caching purposes.

## Repo Naming

Repo names are not always consistent among upstream, cache locations, and subprojects.  When in doubt, refer to `.gitmodules` for guidance.

Expect each subproject of `ansible-roles`, to have an upstream repo name starting with `ansible-role-`.  For example, if you see `ansible-roles/example`, expect the upstream bare repo to be found at `https://github.com/abugher/ansible-role-example.git`.

## Caching Structure

The `cache` directory is expected to contain bare clones of the following repos.  These are not subprojects and should be listed in `.gitignore`.  These are used as a sort of local cache.  Each ansible role contains as a subproject a clone of some number of these.  Some are in all roles, while others are in only a few.  When the subprojects under each role undergo a `git pull` or `git push` operation, it should push or pull to/from the local cache.  That means the local cache needs to regularly sync with any networked upstream repo, but with about a hundred roles I can avoid making about a hundred redundant syncs.

* `ansible-common-tasks.git` - Each role has a copy of this at `tasks/common`.  Shared code to avoid redundant implementations.
* `ansible-environment.git` - Each role has a copy of this at `environment`.  Execution environment for deploying roles to hosts.
* `ansible-inventory.git` - Each role has a copy of this at `inventory`.  My personal inventory.  You probably should replace this with your own.
* `sensitive-ansible-inventory.git` - Not public.  Some roles have a copy of this at `sensitive-inventory`.
* `certificate-authority.git` - Some roles have a copy of this at `certificate-authority`.  Tools for being a CA, operational state of my personal CA, and signed certificates for my hosts.
* `sensitive-certificate-authority.git` - Not public.  Some roles have a copy of this at `sensitive-certificate-authority`.  Sensitive material like private keys for my personal CA.

In order to keep the cache directories synced with the network upstream, there is a working directory corresponding to each one, with the suffix `.sync` instead of `.git`.  Why?  `git` does not like to push from a working repo to a bare repo, so the local cache needs to be bare.  `git` also does not like to pull to a bare repo, so each cache (bare) repo has a corresponding sync (working) repo, which can first pull from the upstream repo (github), then push to the cache.

# Collaboration

## Sharing Roles

You should probably start by making an empty local directory for ansible-roles, which I assume you will name `roles`.  Shop through `ansible-roles` and find a role you want to try, which I will imagine is named `target-role`.  Clone that repo, including any subprojects, to `roles/target-role`.  

    git clone --recurse-submodules https://github.com/abugher/ansible-role-target-role.git roles/target-role

Check `meta` for any dependency relationships to another role, which I will imagine is named `requisite-role`.  Sync it to `roles/requisite-role`.  Repeat as necessary, checking each dependency for further dependencies.

    less roles/target-role/meta/main.yml
    git clone --recurse-submodules https://github.com/abugher/ansible-role-requisite-role.git roles/requisite-role
    less roles/requisite-role/meta/main.yml
    ...

You will probably need to break some things before the role is useful to you.  You will want your own inventory, at a minimum, unless you are replicating my LAN for some reason.  Any subproject with a name starting with `sensitive-` should be unavailable to you, so you will need to create your own.

If you end up using multiple roles, you might want to establish a local cache for some of the subprojects, as described under [Caching Structure](#caching-structure).  Scripts to facilitate this are planned for the near future.

## Sharing Common Tasks

You will probably want the `tasks/common` subproject for any role you try to use.  These roles are mostly composed of references to the tasks defined in that repo.

## Sharing Environment

To make some of the roles work, you may need some of the same ansible configuration I use.  The `environment` suproject under each role contains `ansible.cfg`, providing the configuration, along with some deployment scripts.  You may find the whole subproject useful as is, especially if you want to use my deployment scripts.  Otherwise, you may want to just copy some relevant lines from `ansible.cfg`.

The configuration may not work for you unless modified.  It refers to my inventory, which is probably not valid for you.

## Sharing Inventory

You almost certainly do not want the `inventory` subproject under each role, but you might want to refer to that repo for guidance on writing your own inventory, especially if you plan to use my deployment scripts.

## Sharing Sensitive Information

If the name starts with `sensitive-*`, that means it is not intended to be shared at all.  You're on your own to produce the missing pieces.

## Sharing the Control Center

Cloning `control-center` (this repo) is not recommended.  It has `ansible-roles` as a subproject, which in turn has ALL of my ansible roles as subprojects.  That is a lot.  You probably don't need it all.  Recursive git operations will be slow.

If you insist on trying, first clone this repo:

    git clone https://github.com/abugher/control-center.git control-center

Then check out the branch you want, probably `dev`:

    cd control-center
    git checkout dev

Then run the `populate` script:

    ./bin/populate

It won't work.  You'll probably need to edit the script to refer to your own sources of sensitive information.  It may still not work, since the repos themselves contain submodule definitions referring to my own sources of sensitive information.

## Sharing Difficulty

I may have made collaboration difficult.  You will need to change things to use these roles, as described under [Sharing Roles](#sharing-roles).  If you then make some improvements to a role, it may be difficult to submit a pull request for the improvements while excluding changes that just reflect your different environment.

I will try to improve that situation.

# Tools

## git-automation

The `git-automation` subproject contains `bin/g`.  This is how I manage this hierarchy of repos.  It may be useful for other repos and hierarchies, but it makes some assumptions about how the repos are managed.  Check the documentation for details.

## dicelessware

Password generation.  This may only be used by `bin/generate-host`.

## os-deployment

Before ansible can control a host, an operating system needs to be present.  This contains tools for writing an OS to a boot medium and making initial adjustments to make it accessible enough for ansible to take over.

## bin/generate-host

This is supposed to automate many steps in the initial deployment of a host.

It has not been updated since before a major refactor, so it probably does not work at the moment.  Mostly some paths will need to be updated, I think.

## bin/populate

After cloning this repo non-recursively, I run this script to build the hierarchy of subprojects, install local caches, and adjust remote addresses used for push operations.  Basically, the `--recurse-submodules` option cannot be expected to produce the results I want, so I use this instead.

## bin/fix-remotes

Deprecated.  This crawls through subprojects, finds any remotes on github, and makes sure the push URL uses SSH instead of HTTPS.  It was useful when I was using `git clone --recurse-submodules ...` to install this repo.  Currently its job seems to get done by `bin/populate`.
