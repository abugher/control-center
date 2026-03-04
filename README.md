# Structure

## Branch Structure

These branches are planned:

* `dev`
* `stg`
* `prd`

For each of these branches, I am keeping a separate local instance of this
repo.  Every repo within the hierarchy is on the same branch.  Feature branches
can be created for work on specific goals, but when the goal is complete the
feature branch should be merged into the `dev` branch of the appropriate
repo(s), and the subproject hierarchy of the `dev` branch of `control-center`
should be updated to include the current `dev` branch of the affected repo(s).
This is facilitated by `git-automation/bin/g`.

For now, I just hack on `dev` until whatever I'm working on seems to work, and
then I leave it alone and hack on something else.  Deployments happen from
`dev`, and if they go wrong I keep hacking until they go right.  Not everything
works all the time.

A pipeline is planned.

A testing framework involving virtual machines is in progress.  When that
works, a testing process for each repo will be necessary.  Then, preferably,
the tests should be run automatically when any code is committed to `stg`.
When `stg` passes all tests, it can be synced to `prd`, possibly automatically.
When all that is in place, `prd` will be the branch from which deployments
happen to real systems.

## File Structure

These directories are used in deploying and managing code.  Most are
subprojects.

* `os-deployment` - Tools to apply operating system images to boot media.
* `ansible-environment` - Execution environment for ansible including configuration and deployment scripts.
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

Any directory or repo with a name starting with `sensitive-` should be
unavailable to you.

## Caching Structure

The `cache` directory is expected to contain bare clones of the following
repos.  These are not subprojects and should be listed in `.gitignore`.  These
are used as a sort of local cache.  Each ansible role contains as a subproject
a clone of some number of these.  Some are in all roles, while others are in
only a few.  When the subprojects under each role undergo a `git pull` or `git
push` operation, it should push or pull to/from the local cache.  That means
the local cache needs to regularly sync with any networked upstream repo, but
with about a hundred roles I can avoid making about a hundred redundant syncs.

* `ansible-common-tasks.git` - Each role has a copy of this at `tasks/common`.  Shared code to avoid redundant implementations.

In order to keep the cache directories synced with the network upstream, there
is a working directory corresponding to each one, with the suffix `.sync`
instead of `.git`.  Why?  `git` does not like to push from a working repo to a
bare repo, so the local cache needs to be bare.  `git` also does not like to
pull to a bare repo, so each cache (bare) repo has a corresponding sync
(working) repo, which can first pull from the upstream repo (github), then push
to the cache.

If you end up using multiple roles, you might want to establish a local cache
with similar structure.

# Usage

## Control Center

You will need a control center, but you probably do not want my control center.
You will need to replicate at least some of the directory structure here in
order to make use of my ansible roles, deployment scripts, or other tools.  You
might even name the top level directory `control-center` for simplicity.

Cloning `control-center` (this repo) is not recommended.

### bin/populate

This script will almost certainly not work for you.

After cloning `control-center` non-recursively, I run `bin/populate` to build
the hierarchy of subprojects, install local caches, and adjust remote addresses
used for push operations.  Basically, the `--recurse-submodules` option cannot
be expected to produce the results I want, so I use this instead.

This could use improvement.  It would be nice to be able to use this to repair
the repo if something goes wrong or if `populate` is updated to produces a
slightly different structure, instead of having to create a new clone and
populate it from scratch.  It would also be nice to have a mode of operation in
which `populate` skips site specific repositories like `ansible-inventory` and
`sensitive-*`, so that others could use it to produce an environment similar to
my own.

### bin/fix-remotes

Deprecated.

This crawls through subprojects, finds any remotes on github, and makes sure
the push URL uses SSH instead of HTTPS.  It was useful when I was using `git
clone --recurse-submodules ...` to install this repo.  Currently its job seems
to get done by `bin/populate`.

This script is still useful until `populate` gets a repair mode, at least.

### bin/generate-host

Broken.

This is supposed to automate many steps in establishing a new host.  It writes
components of inventory, bootstraps the host into a valid target for ansible
control, then deploys the roles assigned to the host by group membership in
inventory.

It has not been updated since before a major refactor, so it probably does not
work at the moment.  Mostly some paths will need to be updated, I think.

## OS Deployment

See the [os-deployment repo](../../../os-deployment) for detailed
documentation.

Before ansible can control a host, an operating system needs to be present.
`os-deployment` contains tools for writing an OS to a boot medium and making
initial adjustments to make it accessible enough for ansible to take over.

You can try to clone and use `os-deployment` if you want.  It might be helpful
if you want to run Armbian devices in a similar manner to how I do.

This repo is probably full of site-specific assumptions.  These should be
replaced by references to the inventory where possible.  Pull requests are
welcome.

## Ansible Environment

If you want to deploy any of these roles, you will probably want a copy of
`ansible-environment` in your [control center](#control-center).

### Configuration

`ansible.cfg` may need to be modified to work with your environment.

### Playbooks

`playbooks/deploy.yml`

This generic playbook, consisting mostly of variables, is meant to be called by
the [Deployment Scripts](#deployment-scripts).

### Deployment Scripts

`bin/*`

These scripts launch deployment of roles to hosts.  Any extra arguments after
specified positional arguments will be passed to `ansible-playbook` directly.
Simplest examples:

    deploy-hosts <host[,host][...]|group> [ansible_args]
    deploy-role <role> [ansible_args]

`deploy-hosts` deploys all assigned roles to each host in `group` or a list of
hosts.  If assigned roles depend on further roles those will also be deployed
to the targeted hosts.

`deploy-role` deploys `role` to every host to which `role` is assigned and to
any host to which a role is assigned that depends on `role`, including indirect
dependencies.  If `role` depends on further roles, those will also be deployed
to all targeted hosts.

See [Ansible Inventory](#ansible-inventory) for how to assign a role to a host.

These commands generally expect a remote user named `ansible` with sudo
privileges with no password required.  If the remote host does not yet meet
those requirements, but you have credentials for root or a user with sudo
privileges, you may be able to fix that like so:

    deploy-role-as-user-to-hosts <role> <user> <host[,host][...]|group> [ansible_args]

For example, if you know the password for `root@example`:

    deploy-role-as-user-to-hosts ansible-target root example -k

You can also deploy a specific role to a specific host:

    deploy-role-to-hosts <role> <host[,host][...]|group> [ansible_args]

You can also deploy to localhost, avoiding the need for SSH:

    deploy-role-to-localhost <role> [ansible_args]

Make sure the environment variable `HOSTNAME` is set and matches any inventory
entries.  The script uses that name as the target hostname, so host variables
and group membership for that hostname will be applied.

The `-K` option might be necessary unless the current user account can use
`sudo` with no password.

### Non-Deployment Scripts

`inventory` - See [Dynamic Inventory](#dynamic-inventory) for full discussion.

These options make it a valid dynamic inventory script for use with `ansible` and `ansible-playbook`.

  `inventory --list`
  `inventory --host`

These options expose similarly named internal functions.

    inventory json-basic

Like `--list`, output the statically defined inventory as JSON.

    inventory list-roles
    inventory list-groups

List all roles or all groups.

    inventory list-roles-for-role <role>

List roles depended on by a role including indirect dependencies.

    inventory list-hosts-for-role-explicit <role>
    inventory list-hosts-for-role-implicit <role>
    inventory list-roles-for-host-explicit <host>
    inventory list-roles-for-host-implicit <host>
    inventory list-roles-for-host-tree <host>

List either roles that should be assigned to a host or hosts to which a role is
assigned.  Explicit means the host is assigned the role.  Implicit means the
host is assigned the role or is assigned a role that depends on the role.  Tree
means the information will visually organized according to dependency depth.


## Ansible Inventory

### Dynamic Inventory

Technically `ansible.cfg` specifies a script at
`ansible-environment/bin/inventory` as the dynamic inventory.  That script
regurgitates the [static inventory](#static-inventory) when invoked by ansible
as its dynamic inventory.  It can also be called with one of a number of
arguments to provide listings information beyond what is in the static
inventory, including hosts belonging to groups and roles depending on other
roles.

The functions used in the `inventory` script are also used by the deployment
scripts to make use of information not directly available from the static
inventory, including which hosts are implicitly assigned a role, meaning the
host is part of a hostgroup sharing a name with a role with a dependency
relationship with the role in question.

### Static Inventory

You almost certainly do not want my inventory, but you replicate some of the
structure.  Write your own inventory, and place it in your control center at
`ansible-inventory`, parallel to `ansible-environment`.  The dynamic inventory
script in my [ansible environment](#ansible-environment) and some other code
expect it at that relative path.

#### Role Assignment

The inventory is expected to define hostgroups with the same names as roles.
Any host that is a member of a group with the same name as a role is considered
to be assigned that role.

#### Static Inventory Structure

`inventory/hosts/` contains files specific to hosts, such as certain public
keys.  Contents here should be kept to a minimum.

`inventory/inventory.d/*.yml` are group definition files, one per group, named
for the group.

`inventory/inventory.d/vars` contains global variables.  The domain name and
time zone are set here, for example.

`inventory/inventory.d/host_vars/` contains host variable definition files, one
per host, named for the host, containing details like IP address, MAC address,
and platform.

#### Dashes in Inventory

Group names contain dashes.  Ansible will throw warnings and errors about this
by default, but those should be suppressed if possible.  Be careful, though:
Always refer to `groups['group-name']`, never to `groups.group-name`.  Python
variable names are not allowed to contain dashes.

## Ansible Roles

Cloning the entire `ansible-roles` repo is not recommended, although you
probably could.  Instead, just create a directory named `ansible-roles` in your
control center, parallel to `ansible-environment` if you are using that.  The
configuration in my [ansible environment](#ansible-environment) expects that
relative path for roles.

Shop through `ansible-roles` and find a role you want to try, which I will
imagine is named `target-role`.  Clone that repo, including any subprojects, to
`ansible-roles/target-role`.  

    git clone --recurse-submodules https://github.com/abugher/ansible-role-target-role.git ansible-roles/target-role

Check `meta` for any dependency relationship to another role, which I will
imagine is named `requisite-role`.  Sync it to `ansible-roles/requisite-role`.
Repeat as necessary, checking each dependency for further dependencies.

    less ansible-roles/target-role/meta/main.yml
    git clone --recurse-submodules https://github.com/abugher/ansible-role-requisite-role.git ansible-roles/requisite-role
    less ansible-roles/requisite-role/meta/main.yml
    ...

Each role includes the same set of common tasks at `target-role/tasks/common`.
Most roles consist a list of inclusions of common tasks at
`target-role/tasks/main.yml` and a set of variable definitions at
`target-role/vars/main.yml`.

### Operating System

These roles are written with Debian and a few Debian variants in mind.  The
only package management system is `apt`, unless you count `python` packages.
If you want to apply these roles to a different OS, you will probably need to
modify `install_packages.yml` (under the `tasks/common` subproject in any role)
to use a different package manager.  You may also need to define a slightly
different list of package names in the role variables.  OS-specific paths to
configuration, logs, etc will also need to be defined.  The necessary changes
should be simple but extensive, I expect.

### Dependency Skipping

I have tried to maintain the ability to skip role dependencies when deploying.
This seems reasonable during development because it allows much more rapid
deployments, which is very noticeable when repeatedly writing and testing small
changes.  Currently, every meta file is expected to declare every dependency
with the "dependency" tag.  That's pretty much two identical lines in addition
to every actual dependency line, which really bothers me to look at.  However,
when I need to adjust a configuration file, and the documentation is unclear,
and the configured program is picky, it can be very helpful to try each new
change rapidly.  Like so:

    deploy-role-to-hosts example-role example-host --skip-tags dependency

## Git Automation

`g`:  *Use git as thoughtlessly as possible.*

You can use this if you want to, but you definitely don't have to.  It will
make repetitive operations on nested git repos much more convenient and more
error prone.

Clone the repo anywhere and symlink `bin/g` into your `PATH`.

    g [commit_message]

This command will pull any changes from upstream, then add, commit, and push
any changes from the current working directory and any subdirectories that are
git repos, recursively, depth first.  If a commit message is specified, it will
be applied to all commits; otherwise, git will start an editor for a message
for each commit.

The branch for the top level directory will be checked out for each subproject
and subdirectory.

Collisions will stop the show.  That is probably good, but it is annoying.

Problems may occur when using `g` to sync from an upstream with a new
subproject to a repo previously lacking that subproject.  Further observation
is required to confirm and potentially resolve this issue, but I have not been
trying to use `g` in that way lately, so this is unconfirmed.

# Git Configuration

Just a note on proper global git configuration.  Here is mine, currently:

    user.email=aaron@bugher.net
    user.name=Aaron Bugher
    init.defaultbranch=dev
    commit.gpgsign=true

`commit.gpgsign` means that commits will always be signed, even if `git commit`
is run without the `-S` option.  `g` has `-S` in the standard options for `git
commit`, but sometimes I use `git` directly, and I have a habit of typing `git
commit` with no `-S`.  This makes sure commits get signed anyway.  Before I set
this option (2026-02-26), there are lots of unsigned commits.  Commits should
be consistently signed after this time, at least until I commit from a
different host where I have forgotten to set the option.

I do not (yet) have a good plan for how to ensure this option is set.  I would
like to configure a repo to require signing, preferably by a trusted key,
before accepting a commit.  Github provides options for this, but I would
prefer to make git do this job in place rather than trusting a service to do
it.  Ultimately, I might need my own gitlab instance to accomplish that goal.
