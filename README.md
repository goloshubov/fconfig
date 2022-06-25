Apply one configuration on fedora workstations:
- copy .dotfiles and configs. The files are supposed to be in a separate/private repo - 'filesrepo' inventory var; e.g. the 'all/home' directory files from that repo copied to the user's home (~), the 'all/etc' to the system '/etc/'.
- install software - packages (dnf), flatpaks, pypi (pip), cargo, go packages.
- set some GNOME settings
\
\
Usage:
```bash
$ git clone ...
$ cd wsconfig
$ vim inventory

$ ansible-playbook -i ./inventory apply.yml
or
$ ansible-playbook -i ./inventory apply.yml --list-tags
$ ansible-playbook -i ./inventory apply.yml --tags "tag1, tag2"

```
