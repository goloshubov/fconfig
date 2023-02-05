Ansible playbook that helps keeping one configuration on my fedora linux workstations (and servers).

Features:
- sync .dotfiles and configs
  - The files are supposed to be in separate (and maybe private) git repositories - 'files_repos' inventory variable (an example is https://github.com/goloshubov/fconfig_files). See files_repo's directory structure below.
- install software (from the lists):
  - rpm packages (dnf). 
    - The final package_list is a merge of inventory's (or '{group,host}_vars) package_list and 'union_vars/{group,host}/*' package_list variables. See vars override / merge note below.
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
  - apply dconf key=value settings from the inventory
- TODO: configure other services/apps.

Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
$ vim inventory_workstations.yml        # edit inventory, change ansible_user, files_repos variables
$ mkdir -p ~/git/github/fconfig_files   # create dirs (files_repos.localpath variables)

# apply the configuration:
$ ansible-playbook -i ./inventory_workstations.yml workstation.yml

# or apply it partialy:
$ ansible-playbook -i ./inventory_workstations.yml workstation.yml --list-tags
$ ansible-playbook -i ./inventory_workstations.yml workstation.yml --tags dotfiles
$ ansible-playbook -i ./inventory_workstations.yml workstation.yml --tags packages,flatpaks

```

---
files_repos.repo  (git repository) directory structure:
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
The playbook applies dotfiles in the following order:
```
1. group dir
2. host dir
```

---
Regarding dconf dump files:

The playbook can load dconf dump files from dconf directories (if any). To save dconf dump:
```bash
$ dconf dump <path> > dumpfile
```
expected that dumps are located in the dconf files_repo's directories:
group/<groupname>/dconf/ and byHosname/<hostname>/dconf/
with the file names like '<path>' (used when creating dconf dump files) but where '/' replaced with '__' and without the first and the last '/'\
e.g.
```
$ dconf dump /org/gnome/shell/extensions/dash-to-dock/ > ~/<files_dir>/group/workstation/dconf/org__gnome__shell__extensions__dash-to-dock
```
where filename is: org__gnome__shell__extensions__dash-to-dock. The dump file will be used in the playbook then, to load dconf dump:
```
$ dconf load -f /org/gnome/shell/extensions/dash-to-dock/ < ~/<files_dir>/group/workstation/dconf/org__gnome__shell__extensions__dash-to-dock
```
The playbook applies dconf settings in the following order:
```
1. group dir
2. host dir
3. inventory (dconf_settings)
```

---
vars override / merge note:

'Here is the order of precedence from least to greatest (the last listed variables override all other variables)'
```
...
inventory group_vars/all
inventory group_vars/*
inventory file
inventory host_vars/*
...
```

The union_vars role adds merge option for some variables (package_list):
```
+ include_vars for union_vars/{group,host}/*
```

Meaning that the package_list var from the inventory will be merged with all corresponding union_vars/{group,host}/\<name\>.yml var files, eg:
```
+ union_vars/group/all.yml
+ union_vars/group/workstations.yml
+ union_vars/host/x390yoga.yml
```

This might be usefull to split package list by corresponding group or host names (and to avoid using hash_behaviour).

