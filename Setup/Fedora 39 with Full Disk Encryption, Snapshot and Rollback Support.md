https://www.youtube.com/watch?v=dxTjXnw3Eb0

https://sysguides.com/install-fedora-with-snapshot-and-rollback-support/

# Install requirements

```bash
$ sudo dnf install vim inotify-tools make
```
# Setup Subvolumes

```bash
ROOT_UUID="$(sudo grub2-probe --target=fs_uuid /)"
OPTIONS="$(grep '/home' /etc/fstab | awk '{print $4}' | cut -d, -f2-)"
echo $ROOT_UUID; echo $OPTIONS
# OPTIONS should be like this:
# compress=zstd:1,x-systemd.device-timeout=0

# IMPORTANT: Change the btrf root id according, root or @ 
for dir in "${SUBVOLUMES[@]}" ; do
    if [[ -d "/${dir}" ]] ; then
        sudo mv -v "/${dir}" "/${dir}-old"
        sudo btrfs subvolume create "/${dir}"
        sudo cp -ar "/${dir}-old/." "/${dir}/"
    else
        sudo btrfs subvolume create "/${dir}"
    fi
    sudo restorecon -RF "/${dir}"
    printf "%-41s %-24s %-5s %-s %-s\n" \
        "UUID=${ROOT_UUID}" \
        "/${dir}" \
        "btrfs" \
        "subvol=root/${dir},${OPTIONS}" \
        "0 0" | \
        sudo tee -a /etc/fstab
done

sudo chattr -R +C /var/lib/libvirt/images
sudo lsattr -d /var/lib/libvirt/images
# ---------------C------ /var/lib/libvirt/images

# Review /etc/fstab and mount new volumes

sudo systemctl daemon-reload
sudo mount -va

# Delete old directories
for dir in "${SUBVOLUMES[@]}" ; do
    if [[ -d "/${dir}-old" ]] ; then
        sudo rm -rvf "/${dir}-old"
    fi
done
```

```bash
#### Get the UUID of your btrfs system root.
$ ROOT_UUID="$(sudo grub2-probe --target=fs_uuid /)"

$ echo $ROOT_UUID
ca244df3-0ddb-45c6-bd99-077d9bd81b04

#### Get the btrfs subvolume mount options from your fstab.
$ OPTIONS="$(grep '/home' /etc/fstab | awk '{print $4}' | cut -d, -f2-)"

### Append this to the string: "compress=zstd:1"

$ echo $OPTIONS
# compress=zstd:1,x-systemd.device-timeout=0

#### Create a directory path. Only for other DEs; not required for Fedora Workstation.
$ sudo mkdir -vp /var/lib/libvirt

#### Declare rest of the subvolumes you want to create in the array.
#### Copy from 'SUBVOLUMES' to ')', paste it in terminal, and hit <Enter>.
$ SUBVOLUMES=(
    "opt"
    "var/cache"
    "var/crash"
    "var/log"
    "var/spool"
    "var/tmp"
    "var/www"
    "var/lib/AccountsService"
    "var/lib/gdm"
    "var/lib/libvirt/images"
    "var/lib/docker"
)

$ echo ${SUBVOLUMES[@]}
opt var/cache var/crash var/log var/spool var/tmp var/www var/lib/AccountsService var/lib/gdm var/lib/libvirt/images var/lib/docker

$ for dir in "${SUBVOLUMES[@]}" ; do
    if [[ -d "/${dir}" ]] ; then
        sudo mv -v "/${dir}" "/${dir}-old"
        sudo btrfs subvolume create "/${dir}"
        sudo cp -ar "/${dir}-old/." "/${dir}/"
    else
        sudo btrfs subvolume create "/${dir}"
    fi
    sudo restorecon -RF "/${dir}"
    printf "%-41s %-24s %-5s %-s %-s\n" \
        "UUID=${ROOT_UUID}" \
        "/${dir}" \
        "btrfs" \
        "subvol=root/${dir},${OPTIONS}" \
        "0 0" | \
        sudo tee -a /etc/fstab
done
```
## Fix new directories and delete old ones

```bash
$ sudo chattr -R +C /var/lib/libvirt/images

$ sudo lsattr -d /var/lib/libvirt/images
# ---------------C------ /var/lib/libvirt/images

$ sudo systemctl daemon-reload
$ sudo mount -va

for dir in "${SUBVOLUMES[@]}" ; do
    if [[ -d "/${dir}-old" ]] ; then
        sudo rm -rvf "/${dir}-old"
    fi
done
```

## (Optional) Handle user specific directories

```bash
$ SUBVOLUMES=(
   "home/$USER/.mozilla"
   "home/$USER/.thunderbird"
   "home/$USER/.config/google-chrome"
)
```

# Install Snapper

https://www.youtube.com/watch?v=b-M09CsmfN0

```bash
$ sudo dnf install snapper python3-dnf-plugin-snapper
$ sudo snapper -c root create-config /
$ sudo snapper list-configs
```

## Edit .snapshot's subvolume

```bash
## Change location of subvolume

$ sudo btrfs subvolume delete /.snapshots

$ sudo mkdir /btrfs_pool

$ ls /dev/mapper
# control luks-d1d8cd05-96ca-4a84-8e8e-8633b74d2ca3

$ sudo mount -o subvolid=5 /dev/mapper/luks-d1d8cd05-96ca-4a84-8e8e-8633b74d2ca3 /btrfs_pool

$ sudo btrfs subvolume create /btrfs_pool/.snapshots
$ sudo umount /btrfs_pool
$ sudo mkdir /.snapshots

## Update /etc/fstab to mount the subvolume

## UUID=d0bfa2b8-f788-42be-b39d-5cfe1b9c439d /.snapshots             btrfs   subvol=.snapshots,compress=zstd:1,x-systemd.device-timeout=0 0 0

$ sudo systemctl daemon-reload
$ sudo mount -va
```

# Configure snapshots and grub

```bash
$ sudo snapper -c root set-config ALLOW_USERS=$USER SYNC_ACL=yes
$ echo 'PRUNENAMES = ".snapshots"' | sudo tee -a /etc/updatedb.conf
$ echo 'SUSE_BTRFS_SNAPSHOT_BOOTING="true"' | sudo tee -a /etc/default/grub
$ sudo sed -i '1i set btrfs_relative_path="yes"' /boot/efi/EFI/fedora/grub.cfg
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Install and Configure Grub-Btrfs

https://github.com/Antynea/grub-btrfs

```bash
$ git clone https://github.com/Antynea/grub-btrfs

$ cd grub-btrfs

sed -i '/#GRUB_BTRFS_SNAPSHOT_KERNEL/a GRUB_BTRFS_SNAPSHOT_KERNEL_PARAMETERS="systemd.volatile=state"' config
sed -i '/#GRUB_BTRFS_GRUB_DIRNAME/a GRUB_BTRFS_GRUB_DIRNAME="/boot/grub2"' config
sed -i '/#GRUB_BTRFS_MKCONFIG=/a GRUB_BTRFS_MKCONFIG=/sbin/grub2-mkconfig' config
sed -i '/#GRUB_BTRFS_SCRIPT_CHECK=/a GRUB_BTRFS_SCRIPT_CHECK=grub2-script-check' config

$ sudo make install

$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
$ sudo systemctl enable --now grub-btrfsd.service

```

## Create a System Root Snapshot and Set It as the Default

```bash
$ sudo mkdir -v /.snapshots/1

$ sudo bash -c "cat > /.snapshots/1/info.xml" <<EOF
<?xml version="1.0"?>
<snapshot>
  <type>single</type>
  <num>1</num>
  <date>$(date -u +"%F %T")</date>
  <description>first root subvolume</description>
</snapshot>
EOF

$ sudo btrfs subvolume snapshot / /.snapshots/1/snapshot

$ sudo btrfs inspect-internal rootid /.snapshots/1/snapshot
# 272
$ sudo btrfs subvolume get-default /
# ID 5 (FS_TREE)
$ sudo btrfs subvolume set-default 272 / # Change 290 for the actual ID of the @

$ sudo reboot
```

In case of error in grub boot:
```
rootflags=subvol=root
```

https://docs.fedoraproject.org/en-US/fedora/rawhide/system-administrators-guide/kernel-module-driver-configuration/Working_with_the_GRUB_2_Boot_Loader/#sec-Making_Persistent_Changes_to_a_GRUB_2_Menu_Using_the_grubby_Tool

# Snapper Tests

```bash
$ snapper status 2..3
```

```bash
$ sudo du -sch  /etc /usr
$ sudo rm -rvf /etc /usr
# Choose working snapshot from grub menu
```

# Snaper Usage

https://wiki.archlinux.org/title/snapper#Restoring_/_to_its_previous_snapshot

```bash
$ sudo snapper rollback
Ambit is classic.
Creating read-only snapshot of default subvolume. (Snapshot 4.)
Creating read-write snapshot of current subvolume. (Snapshot 5.)
Setting default subvolume to snapshot 5.
```

reboot your system.

The first thing you should do after a rollback is to update the grub.cfg file so that the GRUB menu snapshots are always in sync.

```bash
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

But first, let me show you how to determine how much space each snapshot takes up on your disk.

```
$ sudo btrfs filesystem du -s --human-readable /.snapshots/*/snapshot
```

first, delete all of the snapshots except #2. You cannot delete snapshot #5 because it is the active one.

```bash
$ snapper delete 1
$ snapper delete 3-4
```

Set one of the snapshots as the new default system root and resume normal operating system use.

```bash
sudo mkdir -v /.snapshots/1
sudo cp -v /.snapshots/2/info.xml /.snapshots/1/
```

Edit the /.snapshots/1/info.xml file and change the content from this...

...to this. Changes are in amber.

```
$ sudo cat /.snapshots/1/info.xml
<?xml version="1.0"?>
<snapshot>
  <type>single</type>
  <num>1</num>
  <date>2023-11-06 08:10:51</date>
  <description>default system root</description>
</snapshot>
```

Create a read-write subvolume snapshot of snapshot #2 in the /.snapshots/1/ directory.

```
$ sudo btrfs subvolume snapshot /.snapshots/2/snapshot /.snapshots/1/snapshot
```

Get the subvolid of the /.snapshots/1/snapshot subvolume.

```
$ sudo btrfs inspect-internal rootid /.snapshots/1/snapshot
277
```

Using subvolid 276, set the /.snapshots/1/snapshot subvolume as the default subvolume for the root (/) filesystem.

```
$ sudo btrfs subvolume set-default 277 /
```

Then reboot.

# Enable Automatic Timeline Snapshots
```bash
$ sudo systemctl enable --now snapper-timeline.timer
$ sudo systemctl enable --now snapper-cleanup.timer
```

## Reference

https://sysguides.com/install-fedora-with-fde-snapshot-rollback-support/

https://mutschler.dev/linux/fedora-btrfs-33/

```bash
$ sudo -i

$ lsblk

$ btrfs subvolume list /
# ID 256 gen 169 top level 5 path home
# ID 258 gen 169 top level 5 path root
# ID 265 gen 24 top level 258 path var/lib/machines

$ blkid

$ ls /dev/mapper
# control luks-e083d283-ceef-44cb-b52d-7bfd17b8afb2

$ mkdir /btrfs_pool

$ mount -o subvolid=5 /dev/mapper/luks-e083d283-ceef-44cb-b52d-7bfd17b8afb2 /btrfs_pool

$ ls /btrfs_pool
# @home @

```

