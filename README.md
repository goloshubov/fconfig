wsconfig is an ansible playbook that initially sets up or updates configuration of (fedora) Linux workstations in an easily reproducible way. It might help to keep the same configuration on a bunch of machines or on the only one. It can
- sync .dot/config files. The files supposed to be in separate/private repo - 'filesrepo' inventory var (the repo's 'files' dir will be synced then with target's '~' dir)
- install apps - packages and flatpaks
- configure GNOME, including setting a desktop wallpaper, selecting favourite apps and their order in the dash/dock.
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim hosts

$ ansible-playbook -i ./hosts apply.yml
```
