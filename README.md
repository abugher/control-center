# Assumptions

## "subproject" vs "submodule"

`man git submodule` refers to repos introduced as the subordinate end of a submodule relationship as `subprojects`.  I do the same.  Some of this might make more sense to you with the word `submodule` where you read `subproject`, depending on how you're used to discussing these things.

## Repo Naming

Repo names are not always consistent among upstream, cache locations, and subprojects.  When in doubt, refer to `.gitmodules` for guidance.

Expect each subproject of `ansible-roles`, to have an upstream repo name starting with `ansible-role-`.  For example, if you see `ansible-roles/example`, expect the upstream bare repo to be found at `https://github.com/abugher/ansible-role-example.git`.

## Operating System

These roles are written with Debian and a few Debian variants in mind.  The only package management system is `apt`, unless you count `python` packages.  If you want to apply these roles to a different OS, you will probably need to modify `install_packages.yml` (under the `tasks/common` subproject in any role) to use a different package manager.  You may also need to define a slightly different list of package names in the role variables.  OS-specific paths to configuration, logs, etc will also need to be defined.  The necessary changes should be simple but extensive, I expect.

# Structure

## Branch Structure

These branches are planned:

* `dev`
* `stg`
* `prd`

For each of these branches, I am keeping a separate local instance of this repo.  Every repo within the hierarchy is on the same branch.  Feature branches can be created for work on specific goals, but when the goal is complete the feature branch should be merged into the `dev` branch of the appropriate repo(s), and the subproject hierarchy of the `dev` branch of `control-center` should be updated to include the current `dev` branch of the affected repo(s).  This is facilitated by `git-automation/bin/g`.

For now, I just hack on `dev` until whatever I'm working on seems to work, and then I leave it alone and hack on something else.  Deployments happen from `dev`, and if they go wrong I keep hacking until they go right.  Not everything works all the time.

A pipeline is planned.

A testing framework involving virtual machines is in progress.  When that works, a testing process for each repo will be necessary.  Then, preferably, the tests should be run automatically when any code is committed to `stg`.  When `stg` passes all tests, it can be synced to `prd`, possibly automatically.  When all that is in place, `prd` will be the branch from which deployments happen to real systems.

## File Structure

These directories are used in deploying and managing code.  Most are subprojects.

* `os-deployment` - Tools to apply operating system images to boot media.
* `ansible-roles` - Full collection of my ansible roles.
* `ansible-roles/*/environment` - Each role has a subproject containing ansible configuration and execution scripts.  See [Caching Structure](#caching-structure).
* `ansible-roles/*/tasks/common` - Each role has a subproject containing a set of commonly used tasks.  See [Caching Structure](#caching-structure).
* `ansible-inventory` - My inventory used mainly by ansible.
* `sensitive-ansible-inventory` - Non-public components of inventory.
* `certificate-authority` - Public components of my personal CA.  Tools, operational state, and signed certificates.
* `sensitive-certificate-authority` - Non public components (eg private keys) of my personal CA.
* `git-automation` - Provides a tool, `g`, used to manage git operations on nested repos like this hierarchy.
* `dicelessware` - Password generation.
* `cache` - Not a subproject.  Ignored by git.  Some repos are cached here to avoid redundant network pulls.  See [Caching Structure](#caching-structure).
* `bin` - Not a subproject.  Scripts for managing and using the contents of `control-center` (this repo).

## Caching Structure

The `cache` directory is expected to contain bare clones of the following repos.  These are not subprojects and should be listed in `.gitignore`.  These are used as a sort of local cache.  Each ansible role contains as a subproject a clone of some number of these.  Some are in all roles, while others are in only a few.  When the subprojects under each role undergo a `git pull` or `git push` operation, it should push or pull to/from the local cache.  That means the local cache needs to regularly sync with any networked upstream repo, but with about a hundred roles I can avoid making about a hundred redundant syncs.

* `ansible-common-tasks.git` - Each role has a copy of this at `tasks/common`.  Shared code to avoid redundant implementations.
* `ansible-environment.git` - Each role has a copy of this at `environment`.  Execution environment for deploying roles to hosts.

In order to keep the cache directories synced with the network upstream, there is a working directory corresponding to each one, with the suffix `.sync` instead of `.git`.  Why?  `git` does not like to push from a working repo to a bare repo, so the local cache needs to be bare.  `git` also does not like to pull to a bare repo, so each cache (bare) repo has a corresponding sync (working) repo, which can first pull from the upstream repo (github), then push to the cache.

# Usage

## Roles

You should probably start by making an empty local directory for ansible-roles, which I assume you will name `ansible-roles`.  Shop through `ansible-roles` and find a role you want to try, which I will imagine is named `target-role`.  Clone that repo, including any subprojects, to `ansible-roles/target-role`.  

    git clone --recurse-submodules https://github.com/abugher/ansible-role-target-role.git ansible-roles/target-role

Check `meta` for any dependency relationships to another role, which I will imagine is named `requisite-role`.  Sync it to `ansible-roles/requisite-role`.  Repeat as necessary, checking each dependency for further dependencies.

    less ansible-roles/target-role/meta/main.yml
    git clone --recurse-submodules https://github.com/abugher/ansible-role-requisite-role.git ansible-roles/requisite-role
    less ansible-roles/requisite-role/meta/main.yml
    ...

Each role includes the same set of common tasks at `target-role/tasks/common`.  Most roles consist a list of inclusions of common tasks at `target-role/tasks/main.yml` and a set of variable definitions at `target-role/vars/main.yml`.

Each role includes the same execution environment at `target-role/environment`.  In concept the configuration could be usable without modification, if you happen to run your systems just like I run mine, but some modification will probably be necessary.

You almost certainly do not want the `inventory` subproject under each role, but you might want to refer to that repo for guidance on writing your own inventory, especially if you plan to use my deployment scripts.  See [Role Assignments](#role-assignments) for assumptions about how inventory should be structured.

Any directory or repo with a name starting with `sensitive-` should be unavailable to you, so if you clone a role referring to one of those, you will need to create your own.

If you end up using multiple roles, you might want to establish a local cache for some of the subprojects, as described under [Caching Structure](#caching-structure).


## Control Center

Cloning `control-center` (this repo) is not recommended.  It has `ansible-roles` as a subproject, which in turn has ALL of my ansible roles as subprojects.  That is a lot.  You probably don't need it all.  Recursive git operations will be slow.

If you insist on trying, first clone this repo:

    git clone https://github.com/abugher/control-center.git control-center

Then check out the branch you want, probably `dev`:

    cd control-center
    git checkout dev

Then run the `populate` script:

    ./bin/populate

It won't work.  You'll probably need to edit the script to refer to your own sources of sensitive information.  It may still not work, since the repos themselves contain submodule definitions referring to my own sources of sensitive information.

## OS Deployment

Before ansible can control a host, an operating system needs to be present.  `os-deployment` contains tools for writing an OS to a boot medium and making initial adjustments to make it accessible enough for ansible to take over.

This repo is probably full of site-specific assumptions.  These should be replaced by references to the inventory where possible.

## bin/generate-host

This is supposed to automate many steps in establishing a new host.  It writes components of inventory, bootstraps the host into a valid target for ansible control, then deploys the roles assigned to the host by group membership in inventory.

It has not been updated since before a major refactor, so it probably does not work at the moment.  Mostly some paths will need to be updated, I think.

## bin/populate

After cloning `control-center` non-recursively, I run `bin/populate` to build the hierarchy of subprojects, install local caches, and adjust remote addresses used for push operations.  Basically, the `--recurse-submodules` option cannot be expected to produce the results I want, so I use this instead.

This could use improvement.  It would be nice to be able to use this to repair the repo if something goes wrong or if `populate` is updated to produces a slightly different structure, instead of having to create a new clone and populate it from scratch.  It would also be nice to have a mode of operation in which `populate` skips site specific repositories like `ansible-inventory` and `sensitive-*`, so that others could use it to produce an environment similar to my own.

## bin/fix-remotes

Deprecated.  This crawls through subprojects, finds any remotes on github, and makes sure the push URL uses SSH instead of HTTPS.  It was useful when I was using `git clone --recurse-submodules ...` to install this repo.  Currently its job seems to get done by `bin/populate`.
