wsconfig is a dotfiles like tool which also configures a (fedora) Linux workstation in an easily reproducible way. It can the following:
- copy .dotfiles and configs. The files are supposed to be in separate/private repo - 'filesrepo' inventory var (the repo's 'home' goes to the ~, and 'etc' to the '/etc/')
- install apps - packages (dnf), flatpaks, pip, cargo, go packages.
- configure GNOME
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim inventory

$ ansible-playbook -i ./inventory apply.yml
```
