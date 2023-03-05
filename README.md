# fconfig
Ansible playbook that helps avoiding configuration drifts on my fedora linux workstations (and servers).

## Features:
- sync .dotfiles and configs
  - files and templates support, recursively (filetree)
  - separate dotfiles git repositories, public or private. An example is https://github.com/goloshubov/fconfig_files See files_repo's directory structure below.
- install software:
  - rpm packages (dnf). 
    - Merge (split) package list support. See vars override / merge note below.
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
- complete package_list var separation: inventory/union_vars/group/all.yml(common) + inventory/group_vars/*(group specific)
- configure other services/apps
- merge support for other package lists (flatpak,pip,cargo,go)
- debian/ubuntu support (?)

## Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
# edit inventory hosts* and variables group_vars/*, union_vars/*

# apply configuration on local workstation (the most used case)
$ ansible-playbook -i ./hosts_workstations workstation.yml -e ansible_connection=local --limit $(hostname)

# or on whole group
$ ansible-playbook -i ./hosts_servers server.yml

# or apply it partialy:
# ansible-playbook -i ./hosts_servers server.yml --list-tags
$ ansible-playbook -i ./hosts_servers server.yml --tags dotfiles
$ ansible-playbook -i ./hosts_servers server.yml --tags packages,flatpaks
$ ansible-playbook -i ./hosts_servers server.yml --tags software --skip-tags cargo

# use bash aliases later (defined in dotfiles: https://github.com/goloshubov/fconfig_files/blob/main/group/all/home/.bashrc.d/aliases.sh)
$ fconfig_local_ws
$ fconfig_local_ws_dotfiles
```

## files repos
files (configs and dotfiles) needs to be located in separate git repositories (one or many). The files_repos variable is defined in inventory/group_vars/all.yml. Here is expected repo directory structure, notice that all supported dirs (dconf, etc, home) are not mandatory:
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

This might be useful to split package list by corresponding group or host files. Then variables will merged during run time.
