wsconfig is an ansible playbook that initially sets up or updates configuration of Linux workstations in an easily reproducible way. It might help to keep the same configuration on a bunch of machines (or on the only one). It can set up .dot files (obviously), and also install apps (packages and flatpaks), configure GNOME (including setting a desktop wallpaper, selecting favorite apps and their order in the dash/dock).
\
\
The only ansible role 'workstation' is wsconfig/roles/workstation/tasks/main.yml. Corresponding dotfiles are in the ./roles/workstation/files/dots/ directory. The default wallpaper is roles/workstation/files/wallpaper.jpg
\
\
Usage:
```bash
$ git clone ...
$ cd dotfiles
$ vim hosts

$ ansible-playbook -i ./hosts apply.yml 
```


