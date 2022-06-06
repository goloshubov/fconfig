wsconfig is a dotfiles like tool which also configures a (fedora) Linux workstation in an easily reproducible way. It can do the following:
- copy .dotfiles and configs. The files are supposed to be in a separate/private repo - 'filesrepo' inventory var; the repo's './home' directory synchronizes with the user's home (~), the './etc' with the system '/etc/'.
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
or
$ ansible-playbook -i ./inventory apply.yml --list-tags
$ ansible-playbook -i ./inventory apply.yml --tags "tag1, tag2"

```
