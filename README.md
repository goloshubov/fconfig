wsconfig is an ansible playbook that initially sets up or updates configuration of (fedora) Linux workstations in an easily reproducible way. It might help keeping the same configuration on a bunch of machines or on the only one. It can:
- copy .dotfiles and configs. The files supposed to be in separate/private repo - 'filesrepo' inventory var (the repo's 'home' dir will be synced then with target's '~' dir, as well as 'etc' with '/etc/')
- install apps - packages (dnf), flatpaks, cargo packages.
- configure GNOME
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim hosts

$ ansible-playbook -i ./inventory apply.yml
```
