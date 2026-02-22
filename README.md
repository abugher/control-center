# Introduction

## "subproject" vs "submodule"

`man git submodule` refers to repos introduced as the subordinate end of a submodule relationship as `subprojects`.  I do the same.  Some of this might make more sense to you with the word `submodule` where you read `subproject`, depending on how you're used to discussing these things.

## Structure

This repo is how I run operations on my home network.  It consists of several
subprojects and some management scripts:

* `ansible-roles` - Full collection of my ansible roles.  If you like one, clone it recursively.  Expect to have to replace some subprojects, like inventory, with your own.
* `dicelessware` - Password generation.
* `git-automation` - Provides a tool, `g`, used to manage git operations on nested repos.
* `os-deployment` - Tools to apply operating system images to boot media.
* `bin` - Not a subproject.  Scripts for managing and using the contents of `control-center` (this repo).

Additionally, this directory is expected to contain bare clones of the following repos.  These are not subprojects and should be listed in `.gitignore`.  These are used as a sort of local cache.  Each ansible role contains as a subproject a clone of some number of these.  Some are in all roles, while others are in only a few.  When the subprojects under each role undergo a `git pull` or `git push` operation, it should push or pull to/from the local cache.  That means the local cache needs to regularly sync with any networked upstream repo, but with about a hundred roles I can avoid making about a hundred redundant syncs.

* `ansible-common-tasks.git` - Each role has a copy of this at `tasks/common`.  Shared code to avoid redundant implementations.
* `ansible-environment.git` - Each role has a copy of this at `environment`.  Execution environment for deploying roles to hosts.
* `ansible-inventory.git` - Each role has a copy of this at `inventory`.  My personal inventory.  You probably should replace this with your own.
* `sensitive-ansible-inventory.git` - Not public.  Some roles have a copy of this at `sensitive-inventory`.
* `certificate-authority.git` - Some roles have a copy of this at `certificate-authority`.  Tools for being a CA, operational state of my personal CA, and signed certificates for my hosts.
* `sensitive-certificate-authority.git` - Not public.  Some roles have a copy of this at `sensitive-certificate-authority`.  Sensitive material like private keys for my personal CA.


# Usage

## Repo Naming

Expect each subproject of `ansible-roles`, to have an upstream repo name starting with `ansible-role-`.  For example, if you see `ansible-roles/example`, expect the upstream bare repo to be found at `https://github.com/abugher/ansible-role-example.git`.

## Partial

You should probably start by making an empty local directory for ansible-roles, which I assume you will name `roles`.  Shop through `ansible-roles` and find a role you want to try, which I will imagine is named `target-role`.  Clone that repo, including any subprojects, to `roles/target-role`.  

    git clone --recurse-submodules https://github.com/abugher/ansible-role-target-role.git roles/target-role

Check `meta` for any dependency relationships to another role, which I will imagine is named `requisite-role`.  Sync it to `roles/requisite-role`.  Repeat as necessary, checking each dependency for further dependencies.

    less roles/target-role/meta/main.yml
    git clone --recurse-submodules https://github.com/abugher/ansible-role-requisite-role.git roles/requisite-role
    less roles/requisite-role/meta/main.yml
    ...

You will probably need to break some things before the role is useful to you.  You will want your own inventory, at a minimum, unless you are replicating my LAN for some reason.  Any subproject with a name starting with `sensitive-` should be unavailable to you, so you will need to create your own.

If you end up using multiple roles, you might want to establish a local cache for some of the subprojects, as described under [Structure](#structure).  Scripts to facilitate this are planned for the near future.


## Full

Cloning `control-center` (this repo) is not recommended.  It has `ansible-roles` as a subproject, which in turn has ALL of my ansible roles as subprojects.  That is a lot.  You probably don't need it all.  Recursive git operations will be slow.  I don't really even have the tooling to manage this hierarchy for myself, at the moment.

# Collaboration

I may have made collaboration difficult.  You will need to change things to use these roles, as described under [Usage](#usage).  If you then make some improvements to a role, it may be difficult to submit a pull request for the improvements while excluding changes that just reflect your different environment.

I will try to improve that situation.
