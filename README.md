wsconfig is enhanced dotfiles written as an ansible playbook that syncs the configuration of (fedora) Linux workstations in an easily reproducible way. It helps to keep the same configuration on a bunch of machines or the only one. It does the following:
- copying .dotfiles and configs. The files are supposed to be in separate/private repo - 'filesrepo' inventory var (the repo's 'home' goes to the ~, and 'etc' to the '/etc/')
- installing apps - packages (dnf), flatpaks, cargo packages.
- configuring GNOME
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim inventory

$ ansible-playbook -i ./inventory apply.yml
```
