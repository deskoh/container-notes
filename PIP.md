# Podman in Podman

## Cache Container Storage for Examples

```sh
mkdir ~/cache ~/cache-root
chmod 777 cache cache-root
```

## Rootful podman

```sh
# Cache:
#   -v ~/cache-root:/var/lib/containers \
#   -v ~/cache:/home/podman/.local/share/containers \

# Rootful Podman in rootful Podman with --privileged
sudo podman run --rm --privileged \
  quay.io/podman/stable podman run alpine id

# Rootful Podman in rootful Podman without --privileged
sudo podman run --rm \
  --cap-add=sys_admin,mknod --device=/dev/fuse --security-opt label=disable \
  quay.io/podman/stable podman run alpine id

```

## Rootless

```sh
# Rootless Podman in rootful Podman with --privileged
podman run --rm --user podman --privileged \
  quay.io/podman/stable podman run alpine id

# Rootless Podman in rootful Podman without --privileged
podman run --rm --user podman \
  --security-opt label=disable --security-opt unmask=ALL --device /dev/fuse \
  quay.io/podman/stable podman run alpine id


# Rootless Podman in rootless Podman
podman run --rm --user podman \
  --security-opt label=disable --device /dev/fuse \
  quay.io/podman/stable podman run alpine id
```

[How to use Podman inside of a container](https://www.redhat.com/sysadmin/podman-inside-container)

[How to use Podman inside of Kubernetes](https://www.redhat.com/sysadmin/podman-inside-kubernetes)