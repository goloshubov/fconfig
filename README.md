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
$ vim inventory.yml

# apply the configuration on the local workstation:
$ ansible-playbook -i ./inventory.yml apply.yml

# or apply it partialy:
$ ansible-playbook -i ./inventory.yml apply.yml --list-tags
$ ansible-playbook -i ./inventory.yml apply.yml --tags packages,flatpaks

```

---
Regarding dconf dump files:

The playbook loads dconf dump files you created before:
```bash
$ dconf dump <path> > dumpfile
```
expected that dumps are located in the files_repo's directories:\
all/home/dconf/, byHosname/\<hostname\>/home/dconf/\
with the file names like '\<path>\' (used when creating dconf dump files) but where '/' replaced with '__' and without the first and the last '/'\
e.g. when filename is:\
org__gnome__shell__extensions__dash-to-dock\
it will be used to load dconf dump by the path of /org/gnome/shell/extensions/dash-to-dock/\
#$ dconf load -f /org/gnome/shell/extensions/dash-to-dock/ < ~/\<files_dir\>/all/home/dconf/org__gnome__shell__extensions__dash-to-dock
\
The playbook applies dconf settings in the following order:
1. dconf dumps, all dir
2. dconf dumps, byHostname dir
3. dconf, inventory (dconf_settings)

---
example files_repo (git repository) dir structure (with some files):
```
.
├── all
│   ├── etc
│   │   ├── sysctl.d
│   │   │   ├── mitigations.conf
│   │   │   └── sysrq.conf
│   │   ├── udev
│   │   │   └── rules.d
│   │   │       └── 51-android.rules
│   │   └── yum.repos.d
│   │       └── kubernetes.repo
│   └── home
│       ├── .ansible.cfg
│       ├── .bashrc
│       ├── bin
│       │   ├── script.sh
│       ├── .config
│       │   ├── helix
│       │   │   └── config.toml
│       │   ├── powerline
│       │   │   └── config.json
│       │   └── xournalpp
│       │       └── toolbar.ini
│       ├── dconf
│       │   └── org__gnome__shell__extensions__dash-to-dock
│       ├── .local
│       │   └── share
│       │       ├── applications
│       │       │   └── scrcpy.desktop
│       │       └── nautilus
│       │           └── scripts
│       │               ├── inHelix
│       │               └── inVim
│       ├── Pictures
│       │   └── wallpapers
│       │       └── wallpaper.jpg
│       ├── .screenrc
│       ├── .tmux.conf
│       ├── .var
│       │   └── app
│       │       └── com.github.xournalpp.xournalpp
│       │           └── config
│       │               └── xournalpp
│       │                   └── toolbar.ini
│       └── .vimrc
├── byHostname
│   ├── ws
│   │   └── etc
│   │       └── tuned
│   │           └── ws-amdgpu
│   │               └── tuned.conf
│   └── x390yoga
│       ├── etc
│       │   ├── default
│       │   │   └── grub
│       │   └── modprobe.d
│       │       └── wacom.conf
│       └── home
```
 chage the inventory.yml vars:
 ```bash
 # git repository with .dotfiles and config
 files_repo: 'git@github.com:goloshubov/wsconfig_files.git'
 files_dir: '~/git/github/wsconfig_files'
```
