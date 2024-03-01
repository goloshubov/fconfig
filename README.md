# fconfig
fconfig is my ansible implementation of dotfiles type of scripts. It helps me avoiding configuration drifts on a bunch of fedora linux workstations and (homelab) servers.

## Features:
- sync .dotfiles and configs
  - files and templates support, recursively (filetree)
  - separate dotfiles git repositories, public or private. An example is https://github.com/goloshubov/fconfig_files See files_repo's directory structure below.
- install software:
  - rpm packages (dnf). 
    - Merge (split) support for list variables (package_list, copr_list). See vars override / merge note below.
    - direct package urls
    - dnf proxy
    - repos configuration
      - enable repos
      - copr repos
      - direct repo urls
      - set mirror for particular repos
  - flatpaks
  - pip packages
  - cargo packages
  - go packages
  - ansible-galaxy collections
- add user to groups
- configure GNOME desktop on workstations 
  - load dconf dump files
  - apply dconf key-value settings

## TODO/consider:
- complete package_list var separation: union_vars/group/all.yml(common) + group_vars/*(group specific)
- configure other services/apps
- merge support for other package lists (flatpak,pip,cargo,go)

## Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
# edit inventory inventories/hosts* and variables group_vars/*, union_vars/*

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


## vars, and the need of vars merge
The problem - defining different package lists for hosts/groups/subgroups.

Options:

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
You have to be really carefull with precedence here. Since you you have to override the vars, there will be a lot of duplicated items in the lists for diffrent hosts/groups. For example, you have to re-define allmost identicall list for child groups here.

Pros: standard approach
Cons: override order can be pure mess

#### 2. dicts and hash_behaviour=merge
Pros: the easiest way
Cons: dicts only, and it's officially not recommended.

#### 3. different var names
To merge diffrent lists one could use additionall tasks or merge them using for example the community.general.lists_union filter.
Pros: No duplication
Cons: Different names. Extra tasks required

union vars naming:
<varname>__union__g_<groupname>
<varname>__union__h_<hostname>
