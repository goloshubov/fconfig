wsconfig is an ansible playbook that initially sets up or updates configuration of Linux workstations in an easily reproducible way. It might help to keep the same configuration on a bunch of machines (or on the only one). It can set up .dot files, and also install apps (packages and flatpaks), configure GNOME (including setting a desktop wallpaper, selecting favorite apps and their order in the dash/dock).
\
\
Usage:
```bash
$ git clone ...
$ cd dotfiles
$ vim hosts

$ ansible-playbook -i ./hosts apply.yml 
```


