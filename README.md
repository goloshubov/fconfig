# fconfig
fconfig is my ansible implementation of dotfiles type of scripts. It helps me avoiding configuration drifts on a bunch of fedora linux workstations and (homelab) servers.

## Features:
- sync .dotfiles and configs
  - files and templates support, recursively (filetree)
  - separate dotfiles git repositories, public or private. An example is https://github.com/goloshubov/fconfig_files See files_repo's directory structure below.
- install software:
  - rpm packages (dnf).
    - Merge (split) support for list variables. See vars override / merge note below.
    - direct package urls
    - dnf proxy
    - repos configuration
      - enable repos
      - copr repos
      - direct repo urls
      - set mirror for particular repos
  - flatpaks
  - pip, pipx packages
  - cargo packages
  - go packages
  - ansible-galaxy collections
- add user to groups
- configure GNOME desktop on workstations
  - load dconf dump files
  - apply dconf key-value settings

## Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
# edit inventory inventories/hosts* and variables group_vars/* host_vars/*

# install prerequisites (fedora)
$ sudo dnf install pipx
# for fedora 41+
$ sudo dnf install python3-libdnf5
$ pipx install --include-deps ansible

# apply configuration on your local workstation (the most used case)
$ ansible-playbook -i ./inventories workstation.yml -e ansible_connection=local --limit $(hostname)

# or apply it partialy, by using tags:
# list all possible tags first
$ ansible-playbook -i ./inventories workstation.yml --list-tags
# optionally dry-run to see what changes to expect
$ ansible-playbook -i ./inventories workstation.yml --tags dotfiles --check --diff -vv
# and then run only what you need
$ ansible-playbook -i ./inventories workstation.yml --tags dotfiles
$ ansible-playbook -i ./inventories workstation.yml --tags software --skip-tags flatpaks
```
```bash
# you can use fconfig to create bash aliases as well, e.g.:
# https://github.com/goloshubov/fconfig_files/blob/main/group/all/home/.bashrc.d/aliases.sh
# and then, my most common use case (just sync dotfiles), it's something like:
$ fconfig_local --tags dotfiles
```

## files repos
files (configs and dotfiles) needs to be located in separate git repositories (one or many). The files_repos variable is defined in inventory/group_vars/all.yml, e.g.:

```yaml
files_repos:
  - { repo: 'git@github.com:username/fconfig_files.git',         localpath: '~/git/github/fconfig_files' }
  - { repo: 'git@github.com:username/fconfig_files_private.git', localpath: '~/git/github/fconfig_files_private' }
  - { repo: 'git@work-git:username/fconfig_files.git',           localpath: '~/git/work/fconfig_files' }
  - { repo: 'git_repo_url', localpath: '<local_clone_path>' }
  - ...
```

Here is expected repo directory structure, notice that all supported dirs (dconf, etc, home) are not mandatory:
```
.
├── group
│   └── <groupname>
│       ├── dconf
│       ├── etc
│       └── home
└── host
    └── <hostname>
        ├── dconf
        ├── etc
        └── home
```
The content of etc/* and home/* dirs will be recursively copied (or templated, for .j2 files) to /etc/* and ~/* dirs respectively on destination hosts according to hostnames and groups.\
Templates are stored with .j2 extension, deployed without it.\
The dconf dir is a store for dconf dump files (GNOME configuration).

The playbook applies dotfiles in the following order:
```
group/all
group/*
host/*
```
An example is https://github.com/goloshubov/fconfig_files

## dconf dump files

The playbook can load dconf dump files from dconf directories (if any). To save dconf dump:
```bash
$ dconf dump <path> > dumpfile
```
expected that dump files are located in the 'dconf' files_repo's directories - 'group/\<groupname\>/dconf/' and 'host/\<hostname\>/dconf/' with file names like '\<path\>' (used when creating dconf dump files) but '/' replaced with '__' (double underscore) and without the first and the last '/'\:
```
$ dconf dump /org/gnome/shell/extensions/dash-to-dock/ > ~/<files_dir>/group/workstation/dconf/org__gnome__shell__extensions__dash-to-dock
```
where filename is: 'org__gnome__shell__extensions__dash-to-dock'. It will be loaded in 'gnome' role:
```
$ dconf load -f /org/gnome/shell/extensions/dash-to-dock/ < ~/<files_dir>/group/workstation/dconf/org__gnome__shell__extensions__dash-to-dock
```
The playbook applies dconf settings in the following order:
```
group/all/dconf/*
group/*/dconf/*
inventory file (dconf_settings)
host/*/dconf/*
```


## variables, merge/overwrite issue.
The problem: to have (for example) package lists for different groups/hosts in group_vars/ and host_vars/i. I see the following options here:
1) Standard merge/override vars way. You define the full list of packages for each group (and host if needed).
Pros: it is clear what packages will be installed, but...
Cons: the main problem here is group precedence (order) and as the result package list vars overwriting (last listed group wins), in case when host is member of several groups; also it might be too many intersecting packages in the lists (not a big deal though).
2) Merge different vars way. You define only specific pakages for each group/host and then a) merge all package list from corresponding groups and host package lists and install it in one task or b) install all correspodning package lists in tasks for each groups and host.
Pros: no precedence issue; only specific packages for group and host in the lists, annd it's clear that if the host is in the group then the group's packages will be definetly installed.
Cons: it might be not that clear what packages (from all corresponding groups and the host) will be installed, you need check all host groups and host's package lists.


More details:
#### 1. Standrad approach and override issue.
According to ansible documentation,
'Here is the order of precedence from least to greatest (the last listed variables override all other variables)'
```
...
group_vars/all
group_vars/*
inventory file
host_vars/*
...
```
You have to be really carefull with precedence here. Since you have to override the vars, there will be a lot of duplicated items in the lists for diffrent hosts/groups. For example, you have to re-define allmost identicall list for child groups here.

Pros: Standard approach.
Cons: Override order can be pure mess.

#### 2. dicts and hash_behaviour=merge
Pros: The easiest way.
Cons: Dicts only, and it's officially not recommended.

#### 3. Extending standard approach by manual overriding
For example by using include_vars with name option + set_fact with it.
Pros: The same soure vars names.
Cons: Custom source vars location. Extra tasks required.

#### 4. different var names
To merge diffrent lists one could use additionall tasks or merge them using for example the community.general.lists_union filter. It's my current implementation, union_vars role.
Pros: No precedence issue and no duplication.
Cons: Different names. Extra task/role required.

1) Add union_vars role in playbook:
2) Define union_vars variable - the list which variables to merge.
3) Defince variables that will be merged:
union_vars naming:
```
<varname>__union__g_<groupname>
<varname>__union__h_<hostname>
```
Meaning that last listed package_list variable, if any, will be merged with all corresponding vars like
```
$ cat group_vars/all.yml
...
package_list__union__g_all:
  - <pcakage1>
  - <package2>
...
```
```
$ cat group_vars/workstations.yml
...
package_list__union__g_workstations:
  - <pcakage3>
  - <package4>
...
```
```
$ cat host_vars/x390yoga.yml
...
package_list__union__h_x390yoga:
  - <pcakage5>
  - <package6>
...
```
