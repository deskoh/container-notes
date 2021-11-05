# Build Recipes

## Use hosts repository cache

```sh
# Run on hosts periodically
dnf -y makecache

# Use overlay mount for dnf cache
buildah bud -v /var/cache/dnf:/var/cache/dnf:O -f Containerfile .

# To see `releasever`:
#  /usr/libexec/platform-python -c 'import dnf, json; db = dnf.dnf.Base(); print(json.dumps(db.conf.substitutions, indent=2))'
# Use subdirectory of /var/cache/dnf to ensure SELinux labels were correct.
dnf -y makecache --releasever=8 --setopt=cachedir=/var/cache/dnf/8
dnf -y makecache --releasever=7 --setopt=cachedir=/var/cache/dnf/7

# To properly clean repo cached files, but some artifacts will remain anyway (gpg keys, empty directories)
dnf clean all --setopt=cachedir=/var/cache/dnf/8
```

## Cache Repo using Containers

```sh
# CentOS 7
mkdir -p /var/cache/centos/7
podman run -v /var/cache/centos/7:/var/cache/yum:z -ti centos:7 yum makecache

# Build container using cache
podman build -v /var/cache/centos/7:/var/cache/yum:O .
```

## References

[Speeding up container image builds with Buildah](https://www.redhat.com/sysadmin/speeding-container-buildah)
