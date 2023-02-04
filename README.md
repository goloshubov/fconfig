Ansible playbook that helps keeping one configuration on fedora linux workstations and servers by doing the following:
- copy .dotfiles, configs. The files are supposed to be in a separate (and maybe private) repos - 'files_repos' inventory variable (an example is https://github.com/goloshubov/fconfig_files). See files_repo's directory sturecture below.
- install software: rpm packages (dnf), flatpaks, pypi (pip), cargo, go, ansible-galaxy collections.
- configure GNOME desktop on workstations (by loading dconf dump files, applying dconf key=value settings from the inventory)
\
\
Usage:
```bash
$ git clone https://github.com/goloshubov/fconfig
$ cd fconfig
# edit inventory, change ansible_user, files_repos variables:
$ vim inventory_workstations.yml
$ mkdir -p ~/git/github/fconfig_files   # files_repos.localpath variables

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
├── all
│   ├── dconf
│   ├── etc
│   └── home
├── byHostname
│   ├── hostname
│   │   ├── dconf
│   │   ├── etc
│   │   └── home
├── byType
│   ├── server
│   │   ├── etc
│   │   └── home
│   ├── workstation
│   │   ├── dconf
│   │   ├── etc
│   │   └── home
```
The playbook applies dotfiles in the following order:
1. all dir
2. byType dir
3. byHostname dir

---
Regarding dconf dump files:

The playbook can load dconf dump files you created before:
```bash
$ dconf dump <path> > dumpfile
```
expected that dumps are located in the files_repo's directories:\
all/dconf/, byType/\<type\>/dconf/ and byHosname/\<hostname\>/dconf/\
with the file names like '\<path>\' (used when creating dconf dump files) but where '/' replaced with '__' and without the first and the last '/'\
e.g.\
$ dconf dump /org/gnome/shell/extensions/dash-to-dock/ > ~/\<files_dir\>/all/dconf/org__gnome__shell__extensions__dash-to-dock\
where filename is: org__gnome__shell__extensions__dash-to-dock\
the dump file will be used in the playbook then, to load dconf dump:\
$ dconf load -f /org/gnome/shell/extensions/dash-to-dock/ < ~/\<files_dir\>/all/dconf/org__gnome__shell__extensions__dash-to-dock\
\
The playbook applies dconf settings in the following order:
1. dconf dumps, all dir
2. dconf dumps, byType dir
3. dconf dumps, byHostname dir
4. dconf, inventory (dconf_settings)


