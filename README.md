wsconfig is an ansible playbook that helps keeping one configuration on your (multiple) fedora workstations (desktops), by doing the following:
- copy .dotfiles, configs. The files are supposed to be in a separate/private repo - 'files_repo' inventory variable. files_repo's sturecture is "all/{etc,home}/" and "byHostname/\<hostname\>/{etc,home}/", e.g. the 'all/home' directory files from that repo copied to the user's home (~), the 'all/etc' to the system '/etc/'.
- install software: rpm packages (dnf), flatpaks, pypi (pip), cargo, go packages.
- change GNOME configuration (by loading dconf dump files and applying dconf key=value settings from the inventory)
\
\
Usage:
```bash
$ git clone https://github.com/goloshubov/wsconfig
$ cd wsconfig
# edit inventory, edit ansible_user, files_repo and files_dir variables:
$ vim inventory

# apply the configuration on the local workstation:
$ ansible-playbook -i ./inventory.yml apply.yml

# or apply it partialy:
$ ansible-playbook -i ./inventory.yml apply.yml --list-tags
$ ansible-playbook -i ./inventory.yml apply.yml --tags packages,flatpaks

```

Regarding GNOME:
The playbook will load dconf setting using dconf dump files you created earlier
```bash
$ dconf dump <path> > dumpfile
```
expected that dump files are located in the files_repo's directory:
all/home/dconf/ or byHosname/<hostname>/home/dconf/
with the file names like '<path>' (you used when creating dconf dump files) but where '/' replaced with '__' and withoud first and last '/'
e.g. when filename is:
org__gnome__shell__extensions__dash-to-dock
it will be used to load dconf by the path of /org/gnome/shell/extensions/dash-to-dock/
```bash
#playbook load dump files like#$ dconf load -f <path> < dumpfile
```
dump files loaded before dconf_settings, order:
1. dconf dumps, all
2. dconf dumps, byHostname
3. dconf from dconf_settings var (might overwrite what were laoded with dump files)
