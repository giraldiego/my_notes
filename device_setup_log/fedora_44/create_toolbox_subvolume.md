https://www.tecmint.com/fedora-toolbx-containerized-development/

https://containertoolbx.org/install/

Check the locations:

```
podman info --format '{{.Store.GraphRoot}} {{.Store.RunRoot}}'
```

# 1. Stop Podman
podman stop --all

# 2. Move existing data
mv ~/.local/share/containers/storage ~/.local/share/containers/storage-old

# 3. Create the subvolume
btrfs subvolume create ~/.local/share/containers/storage

# 4. Copy data back
cp -a ~/.local/share/containers/storage-old/. ~/.local/share/containers/storage/

# 5. Verify, then clean up
podman images
rm -rf ~/.local/share/containers/storage-old


