# fconfig
fconfig is my ansible playbook to configure linux workstation

## goals
- avoid configuration drifts on your machines. Meaning - switching to another laptop without noticing the difference
- keep it as simple as possible

## features
- apply dotfiles and system configs:
  - plain files and jinja templates are supported
  - configs stored in (external) git repositories, public or private. files_repo's directory structure described below
- install software:
  - fedora. rpm packages (dnf).
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
- configure GNOME desktop:
  - load dconf dump files
  - apply dconf key-value settings

## usage
```bash
# Usually (when bash aliases already applied) I just run smth like
$ fconfig_local --tags dotfiles
# or
$ fconfig_local --tags dotfiles,gnome
```

```bash
# on a new machine
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

## files repos
files (configs and dotfiles) needs to be located in separate git repositories (one or many). The files_repos variable is defined in inventory/group_vars/all.yml, e.g.:

```yaml
files_repos:
  - { repo: 'git@github.com:username/fconfig_files.git',         localpath: '~/git/github/fconfig_files' }
  - { repo: 'git@github.com:username/fconfig_files_private.git', localpath: '~/git/github/fconfig_files_private' }
  - { repo: 'git@work-git:username/fconfig_files.git',           localpath: '~/git/work/fconfig_files' }
  - { repo: 'git_repo_url', localpath: '<controller_local_path>' }
  - ...
```

Here is expected repo directory structure, notice that all supported dirs (dconf, etc, home) are not mandatory:
```
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
dump files supposed to be located in the 'files_repo' directories, where 'group/\<groupname\>/dconf/' and 'host/\<hostname\>/dconf/' contain files with names like dconf '\<path\>' but '/' replaced with '__' (double underscore) and without the first and the last '/'\, e.g. 'org__gnome__shell__extensions__dash-to-dock'.
```
$ dconf dump /org/gnome/shell/extensions/dash-to-dock/ > ~/<files_dir>/group/workstation/dconf/org__gnome__shell__extensions__dash-to-dock
```
It will be loaded in 'gnome' role:
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

## TODOs
- distrs support: Debian, Ubuntu, ALT Linux
- import (additional) package lists from files (requirement lists)
- additional roles for services with restart notifcations support. It might be usefull even for workstations/laptops, on other hand it will add some complexity.
