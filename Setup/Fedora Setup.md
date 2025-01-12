# Installation & Essentials

## Shortcuts
### gnome-terminal
```
1. Open the Settings app.
2. Click on Keyboard.
3. Click on View and Customize Shortcuts.
4. Click on the Custom Shortcuts tab.
5. Click on the + button to add a new shortcut.
6. In the Name field, type Launch Terminal.
7. In the Command field, type `ptyxis` (replaces gnome-terminal).
8. In the Shortcut field, type Super+T.
9. Click on the Add button.
```
### gnome-text-editor
```
6. In the Name field, type Text Editor.
7. In the Command field, type `gnome-text-editor`.
8. In the Shortcut field, type Super+E.
9. Click on the Add button.
```
## Things to do after installing Fedora

https://itsfoss.com/things-to-do-after-installing-fedora/
https://arcanesavant.github.io/p/2024/10/31/12-things-to-do-after-installing-fedora-workstation-41/
https://fosspost.org/things-to-do-after-installing-fedora/


``` bash
## 1. Configure DNF for Faster Downloads of Packages

sudo bash -c 'cat >> /etc/dnf/dnf.conf' <<EOF
max_parallel_downloads=10
EOF

## 3. Enable RPM Fusion Repository 

sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

## 6. Change Hostname After Installation

sudo hostnamectl set-hostname "fedora"

```

### Video codecs

```bash
## 5. Install Multimedia Plugins

sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} gstreamer1-plugin-openh264 gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel

sudo dnf install lame\* --exclude=lame-devel

sudo dnf group upgrade --with-optional Multimedia
```

```bash
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf install -y gstreamer1-plugin-openh264 mozilla-openh264
```

### Firefox Video Cannot Play 

https://www.youtube.com/watch?v=-lxG8TGmR_o

https://www.reddit.com/r/Fedora/comments/1354e2u/firefox_video_cannot_play/

https://rpmfusion.org/Howto/Multimedia

```bash
## Switch to full ffmpeg
sudo dnf swap ffmpeg-free ffmpeg --allowerasing

## Install additional codec
sudo dnf groupupdate multimedia --setop="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
sudo dnf groupupdate sound-and-video

## Hardware codecs with AMD (mesa)
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```
# Setup and Configurations
## SSH Keys
https://docs.github.com/authentication/connecting-to-github-with-ssh
```bash
ssh-keygen -t ed25519 -C "giraldiego@gmail.com"

eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_ed25519
```

## BTRFS

[[Fedora 39 with Full Disk Encryption, Snapshot and Rollback Support]]
### btrfs-luks full disk encryption
https://mutschler.eu/linux/install-guides/fedora-btrfs-33/
https://mutschler.eu/linux/install-guides/fedora-post-install/
### Setup auto snapshots with Timeshift
https://mutschler.eu/linux/install-guides/fedora-btrfs-33/
### BTRBK
https://lukas.zapletalovi.com/posts/2022/fast-backups-of-fedora-with-btrbk/
https://mutschler.eu/linux/install-guides/fedora-btrfs-35/

## NVIDIA

https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/

https://www.linuxcapable.com/how-to-install-nvidia-drivers-on-fedora-linux/

```bash
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora37/x86_64/cuda-fedora37.repo

sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora36/x86_64/cuda-fedora36.repo

sudo dnf install kernel-headers kernel-devel tar bzip2 make automake gcc gcc-c++ pciutils elfutils-libelf-devel libglvnd-opengl libglvnd-glx libglvnd-devel acpid pkgconfig dkms

sudo dnf module list nvidia-driver
```


## Dropbox

https://www.linuxshelltips.com/install-dropbox-fedora-linux/

https://www.dropbox.com/install

````bash
$ sudo dnf install libgnome libgnome-devel -y
$ sudo dnf install ./nautilus-dropbox-2023.09.06-1.fedora.x86_64.rpm
````
## Gnome Extensions

### AppIndicator and KStatusNotifierItem Support
https://extensions.gnome.org/extension/615/appindicator-support/

>Do not use this version from extensions.gnome.org – it lacks libappindicator-gtk3, libdbusmenu(-gtk3) and other dependencies needed for some icons to display properly. Use the system extension instead if possible. Here is a guide for Fedora users:
>
> 1. Install the extension. Open a Terminal and enter:
> `sudo dnf install gnome-shell-extension-appindicator`
> 2. Relog or reboot your computer.
> 3. Enable the extension. Open a Terminal and enter:
> gnome-extensions enable appindicatorsupport@rgcjonas.gmail.com
> 4. Enjoy full tray icons support!
> 
> P.S. Consider supporting the upstream initiative that will hopefully replace this extension in near future:
> https://gitlab.freedesktop.org/xdg/xdg-specs/-/merge_requests/54
> https://gitlab.gnome.org/Teams/Design/os-mockups/-/issues/191

### gTile
https://extensions.gnome.org/extension/28/gtile/
https://github.com/gTile/gTile#install-from-source

## Oh-My-Bash

https://ohmybash.nntoan.com/
https://www.youtube.com/watch?v=v-IjFE34M8s

[Customization of Plugins and Themes](https://github.com/ohmybash/oh-my-bash#customization-of--plugins-and-themes)

`~/.oh-my-bash/custom/themes/powerline/`

`OMB_PROMPT_SHOW_PYTHON_VENV=true`
### Don't show absolute path in prompt
```bash
function __powerline_cwd_prompt {
  # echo "$(pwd | sed "s|^${HOME}|~|")|${CWD_THEME_PROMPT_COLOR}"
  echo "$(pwd | rev | cut -d '/' -f 1 | rev | sed "s|^${HOME}|~|")|${CWD_THEME_PROMPT_COLOR}"
}
```
[source](https://superuser.com/questions/60555/show-only-current-directory-name-not-full-path-on-bash-prompt)

### Show pyenv virtualenv in prompt
```bash
function __powerline_python_venv_prompt {
  local python_venv=""

  if [[ -n "${CONDA_DEFAULT_ENV}" ]]; then
    python_venv="${CONDA_DEFAULT_ENV}"
    PYTHON_VENV_CHAR=${CONDA_PYTHON_VENV_CHAR}
  elif [[ -n "${VIRTUAL_ENV}" ]]; then
    python_venv=$(basename "${VIRTUAL_ENV}")
  # for pyenv-virtualenv
  elif [[ -n $PYENV_SHELL ]]; then
    version=$(pyenv version | cut -f 1 -d\()
    if [[ $version != "system " ]]; then
      python_venv=$(basename "${version}")
    fi
  # end
  fi
```
[source](https://gist.github.com/loganasherjones/bd9b7614f80b96cf700fd60e9e256f41)

# Libvirt

https://www.youtube.com/watch?v=BgZHbCDFODk

https://docs.fedoraproject.org/en-US/quick-docs/virtualization-getting-started/

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
$ sudo dnf install @virtualization
$ sudo systemctl start libvirtd
$ sudo systemctl enable libvirtd
```

## Bluetooth

https://www.howtogeek.com/829360/how-to-set-up-bluetooth-on-linux/

# Custom Config

## App drawer shortcuts

https://askubuntu.com/questions/747118/where-is-the-location-of-application-shortcuts-in-gnome

You are looking for the `.desktop` files, which are located in `/usr/share/applications` (or sometimes `/usr/local/share/applications`).

Some applications may place these files in `~/.local/share/applications` instead.

1.  **Most apps:** /usr/share/applications/
2.  **Some apps:** /usr/local/share/applications
3.  **Some other apps:** ~/.local/share/applications/
4.  **Installed with Snap apps:** /var/lib/snapd/desktop/applications/
5.  **Installed with Flatpak apps:** /var/lib/flatpak/exports/share/applications/
6.  **Installed with Wine apps:** ~/.local/share/applications/wine/Programs/

## Disable ibus shortcut

https://stackoverflow.com/questions/71997823/ctrl-dot-makes-e-appear-instead-of-showing-suggestions-in-vscode-on-gnome

`$ ibus-setup`

## Disable gnome keybinds

https://unix.stackexchange.com/questions/394143/how-to-disable-gnome-ctrlaltdown-and-ctrlaltup-shortcut

```bash
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-up "['']"

gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-down "['']"
```


# Add ONs

## Docker

https://docs.docker.com/engine/install/fedora/
https://docs.docker.com/engine/install/linux-postinstall/
https://docs.docker.com/engine/security/rootless/

https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd

https://docs.docker.com/engine/install/fedora/#set-up-the-repository

https://docs.docker.com/desktop/install/fedora/

```bash
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```

https://www.baeldung.com/ops/docker-image-change-installation-directory

https://linuxconfig.org/how-to-move-docker-s-default-var-lib-docker-to-another-directory-on-ubuntu-debian-linux
### Docker Compose
https://docs.docker.com/compose/migrate/

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## VirtualBox

https://tecadmin.net/install-oracle-virtualbox-on-fedora/

## Installing Snap
https://www.cyberciti.biz/faq/install-snapd-on-fedora-linux-system-dnf-command/

## Calibre

Favorites:
https://manual.calibre-ebook.com/sub_groups.html

ReadingLists:
https://www.mobileread.com/forums/showthread.php?t=134856

## AppImages

https://fedoraproject.org/wiki/AppImage

https://github.com/TheAssassin/AppImageLauncher

## ADB

https://linuxconfig.org/how-to-use-adb-android-debug-bridge-to-manage-your-android-mobile-phone

https://github.com/0x192/universal-android-debloater

## rclone

https://rclone.org/downloads/

```bash
$ sudo -v ; curl https://rclone.org/install.sh | sudo bash
$ sudo dnf install rclone-browser

```

```bash
rclone listremotes
```

https://tutorialforlinux.com/2023/04/18/step-by-step-rclone-browser-fedora-38-installation-guide/2/

## Onedrive

https://www.linuxuprising.com/2020/02/how-to-keep-onedrive-in-sync-with.html

https://github.com/abraunegg/onedrive/tree/master

```bash
### Dependencies: Fedora > Version 18 / CentOS 8.x / RHEL 8.x / RHEL 9.x
sudo dnf groupinstall 'Development Tools'
sudo dnf install libcurl-devel sqlite-devel
curl -fsS https://dlang.org/install.sh | bash -s dmd

### **Note:** Depending on your DMD version, substitute `2.088.0` above with your DMD version that is installed.

git clone https://github.com/abraunegg/onedrive.git
cd onedrive
./configure
make clean; make;
sudo make install
```

## SageMaker Studio (Docker)
https://github.com/aws/sagemaker-distribution
https://gallery.ecr.aws/sagemaker/sagemaker-distribution

```bash
export ECR_IMAGE_ID='INSERT_IMAGE_YOU_WANT_TO_USE'
docker run -it \
    -p 8888:8888 \
    --user `id -u`:`id -g` \
    -v `pwd`/sample-notebooks:/home/sagemaker-user/sample-notebooks \
    $ECR_IMAGE_ID jupyter-lab --no-browser --ip=0.0.0.0
```



## qemu-kvm

https://computingforgeeks.com/how-to-install-kvm-on-fedora/
## lxd

https://www.cyberciti.biz/faq/how-to-install-setup-lxd-on-fedora-linux/

# Backup

### Restore from Backup[](https://mutschler.dev/linux/fedora-post-install/#restore-from-backup)

I mount my LUKS encrypted backup storage drive using nautilus (simply click on it in the file manager). Then let’s use rsync to copy over my files and important configuration scripts:

```sh
export BACKUP=/run/media/$USER/NAME_OR_UUID_BACKUP_DRIVE/@home/$USER/
sudo rsync -avuP $BACKUP/Desktop ~/
sudo rsync -avuP $BACKUP/Documents ~/
sudo rsync -avuP $BACKUP/Downloads ~/
sudo rsync -avuP $BACKUP/Music ~/
sudo rsync -avuP $BACKUP/Pictures ~/
sudo rsync -avuP $BACKUP/Templates ~/
sudo rsync -avuP $BACKUP/Videos ~/
sudo rsync -avuP $BACKUP/.ssh ~/
sudo rsync -avuP $BACKUP/.gnupg ~/

sudo rsync -avuP $BACKUP/.local/share/applications ~/.local/share/
sudo rsync -avuP $BACKUP/.gitconfig ~/
sudo rsync -avuP $BACKUP/.gitkraken ~/
sudo rsync -avuP $BACKUP/.config/Nextcloud ~/.config/

sudo rsync -avuP $BACKUP/dynare ~/
sudo rsync -avuP $BACKUP/.dynare ~/
sudo rsync -avuP $BACKUP/Images ~/
sudo rsync -avuP $BACKUP/SofortUpload ~/
sudo rsync -avuP $BACKUP/Work ~/
sudo rsync -avuP $BACKUP/Zotero ~/
sudo rsync -avuP $BACKUP/MATLAB ~/
sudo rsync -avuP $BACKUP/.matlab ~/

sudo chown -R $USER:$USER /home/$USER # make sure I own everything
```

# Hardware
## Mouse latency

https://winaero.com/fix-bluetooth-mouse-lag-on-linux-for-device-without-transmitter/

## Optimize SD Card

https://developer.ridgerun.com/wiki/index.php/High_performance_SD_card_tuning_using_the_EXT4_file_system

```bash
sudo umount /dev/sda1
sudo tune2fs -o journal_data_writeback /dev/sda1
sudo tune2fs -O ^has_journal /dev/sda1
sudo e2fsck -f /dev/sda1
```


You can check your filesystem setup using:

`$ sudo dumpe2fs /dev/sdb1 | less`

Mount the EXT4 partition specifying the following flags for optimization:

- noatime - avoid writing access times
- data=writeback - tell mount we are mounting a filesystem with no journaling

To mount the EXT4 fileystem at /mnt/sd in your board:
```bash
mkdir -p /mnt/sd
mount -t ext4 -O noatime,data=writeback /dev/mmcblk0p1 /mnt/sd
```