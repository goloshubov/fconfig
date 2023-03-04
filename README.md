# fconfig
Ansible playbook that helps avoiding configuration drifts on my fedora linux workstations (and servers).

## Features:
- sync .dotfiles and configs
  - The files are supposed to be in separate (and maybe private) git repositories - 'files_repos' variable (an example is https://github.com/goloshubov/fconfig_files). See files_repo's directory structure below.
- install software:
  - rpm packages (dnf). 
    - Merge (split) package list support. See vars override / merge note below.
    - direct package URL
    - enable repos
    - copr repos
    - direct repo URL
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
- complete package_list var separation: inventory/union_vars/group/all.yml(common) + inventory/group_vars/*(group specific)
- configure other services/apps
- debian/ubuntu support
- merge support for other package lists (flatpak,pip,cargo,go)

## Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
# edit inventory in inventory/hosts* and variables in inventory/group_vars/*, inventory/union_vars/*
$ mkdir -p ~/git/github/fconfig_files   # create dirs (files_repos.localpath variables)

# apply configuration on local workstation
$ ansible-playbook -i ./inventory workstation.yml -e ansible_connection=local --limit $(hostname)

# or on whole group
$ ansible-playbook -i ./inventory workstation.yml

# or apply it partialy:
# ansible-playbook -i ./inventory workstation.yml --list-tags
$ ansible-playbook -i ./inventory workstation.yml --tags dotfiles
$ ansible-playbook -i ./inventory workstation.yml --tags packages,flatpaks
$ ansible-playbook -i ./inventory workstation.yml --tags software --skip-tags cargo

# or just use bash aliases later (defined in dotfiles: https://github.com/goloshubov/fconfig_files/blob/main/group/all/home/.bashrc.d/aliases.sh)
$ fconfig_local_ws
$ fconfig_local_ws_dotfiles
```

## files repos
files (configs and dotfiles) needs to be located in separate git repositories (one or many). The files_repos variable is defined in inventory/group_vars/all.yml. Here is expected repo directory structure, notice that all supported dirs (dconf, etc, home) are not mandatory:
```
.
├── group
│   ├── <groupname>
│   │   ├── dconf
│   │   ├── etc
│   │   └── home
├── host
│   ├── <hostname>
│   │   ├── dconf
│   │   ├── etc
│   │   └── home
```
The etc/* and home/* dirs will be synced (one way) with /etc/* and ~/* dirs on a appropriate machine (according to its hostname and groupname).\
The dconf dir is a store for dconf dump files (GNOME configuration).

The playbook applies dotfiles in the following order:
```
group/all
group/*
host/*
```

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

The union_vars role adds merge option for some variables (package_list for now).
Meaning that the final package_list variable will be a merge of last listed variable from group_var or inventory and corresponding variables from 'union_vars/{group,host}/*
```
+ union_vars/group/all.yml
+ union_vars/group/<groupname>.yml
+ union_vars/host/<hostname>.yml
```

This might be usefull to split package list by corresponding group or host names (and to avoid using hash_behaviour).

