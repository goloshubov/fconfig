wsconfig is an ansible playbook that helps keeping one configuration on your (multiple) fedora workstations (desktops), by doing the following:
- copy .dotfiles, configs. The files are supposed to be in a separate/private repo - 'filesrepo' inventory. filesrepo's sturecture is "{all,\<hostname\>}/{etc,home}/", e.g. the 'all/home' directory files from that repo copied to the user's home (~), the 'all/etc' to the system '/etc/'.
- install software: rpm packages (dnf), flatpaks, pypi (pip), cargo, go packages.
- set GNOME settings
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim inventory

# apply the configuration on the local workstation:
$ ansible-playbook -i ./inventory apply.yml

# or apply configuration partialy
$ ansible-playbook -i ./inventory apply.yml --list-tags
$ ansible-playbook -i ./inventory apply.yml --tags "tag1, tag2"

```
