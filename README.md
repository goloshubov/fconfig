# fconfig
fconfig is my ansible playbook to configure a (fedora) linux workstation

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
  - pip, pipx packages, virtualenvs
  - cargo packages
  - go packages
  - ansible-galaxy collections
- add user to groups
- configure GNOME desktop:
  - load dconf dump files
  - apply dconf key-value settings

## usage
```bash
# Usually (when dotfiles and bash aliases in it are already applied) I just run smth like
$ fconfig_local --tags dotfiles
# or
$ fconfig_local --tags dotfiles,gnome
```

```bash
# on a new machine

# install ansible
# venv way (preferable)
$ mkdir ~/python-venv
$ cd !$
$ python3 -m venv ansible
$ source ansible/bin/activate
$ python3 -m pip install --upgrade pip
$ python3 -m pip install ansible
$ ansible --version
## pipx way
#$ sudo dnf install pipx
## for fedora 41+
#$ sudo dnf install python3-libdnf5
#$ pipx install --include-deps ansible
# package way of ansible installation is not recommended, possible though

$ mkdir -p ~/git/github
$ cd !$
$ git clone https://github.com/goloshubov/fconfig   # or fork
$ cd fconfig
# - edit inventory and variables (inventories/, group_vars/, host_vars/),
#   do not use file extension for inventories (in order to be able to use the whole directory)
# - commit changes, and optionaly move it to your own repository if not already
# - prepare your dotfiles repository, one or many
#   see 'files repos' section in README for details, as an example https://github.com/goloshubov/fconfig_files

# complete ansible installation,
# for venv ansible installation by setting up ansible venv with requirements that will found in
# {{ python-venv-dir }}/ansible.requirements.txt after tasks with dotfiles tag applied
$ ansible-playbook -i ./inventories workstation.yml -e ansible_connection=local \
    --limit $(hostname) --tags dotfiles,pyvenv
## for pipx ansible installation
#$ ansible-playbook -i ./inventories workstation.yml -e ansible_connection=local --limit $(hostname) --tags dotfiles,pipx
## for package way of ansible installation it might be needed to install additionall dependencies (see playbook output in case any errors)

# from now on it is possible to use short fconfig_local alias
# for example apply all configs to your local workstation
$ alias fconfig_local  # check alias, change it if needed
$ fconfig_local

# or apply it partially
# list all possible tags first
$ fconfig_local --list-tags
# optionally dry-run to see what will be changed
$ fconfig_local --tags dotfiles --check --diff -vv
# and then run only what you need
$ fconfig_local --tags dotfiles
$ fconfig_local --tags software --skip-tags flatpaks

# or apply configuration to a remote machine
$ ansible-playbook -i ./inventories workstation.yml --limit laptop-02 --tags dotfiles
```

## files repos
Note that is possible to keep playbook/roles and dotfiles/configs together in one (and the only) repository, in different repos, or combination of both.
It is recommened that playbook and files (configs and dotfiles) are located in separate git repositories.
The files_repos variable migth be defined in inventory/group_vars/all.yml, e.g.:

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
- add other distrs support: Debian, Ubuntu, ALT Linux
- import (additional) package lists from files (requirement lists)
- additional roles for services with restart notifcations support. It might be usefull even for workstations/laptops, on other hand it will add some complexity.
- ansible-pull automation
