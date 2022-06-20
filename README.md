# Containers Notes

## WSL2

```sh
# verify configuration for WSL2
podman info

# /usr/share/containers/containers.conf
events_logger = "file"
cgroup_manager = "cgroupfs"
network = "host"
```

## Storage and Config locations

Root user: `/etc/containers`, `/var/lib/containers`

Non-root user: `~/.config/containers/`, `~/.local/share/containers/`

Short names are cached in `~/.cache/containers/short-name-aliases.conf`.

CA certs should be stored in `/etc/docker/certs.d/{registry url}`

## Podman search behavoirs

1. Look for credentials

1. Look for certs and private keys in `/etc/docker/certs.d/<registry url>

1. Look for v1 and v2 HTTPS / HTTP endpoints that return `401`

```txt
DEBU[0000] Credentials not found
DEBU[0000] Looking for TLS certificates and private keys in /etc/docker/certs.d/p.cr.io
DEBU[0000] trying to talk to v1 search endpoint
DEBU[0000] GET https://p.cr.io/v2/
DEBU[0000] Ping https://p.cr.io/v2/ status 404
DEBU[0000] GET http://p.cr.io/v2/
DEBU[0000] Ping http://p.cr.io/v2/ status 401
DEBU[0000] GET http://p.cr.io/v1/search?n=25&q=node
DEBU[0000] error getting search results from v1 endpoint "p.cr.io": invalid status code from registry 400 (Bad Request)
DEBU[0000] trying to talk to v2 search endpoint
DEBU[0000] GET http://p.cr.io:80/artifactory/api/docker/docker-private/v2/token?service=p.cr.io%3A80
DEBU[0000] GET http://p.cr.io/v2/_catalog
```

## Podman pull behavoirs

> Container storage: `~/.local/share/containers/`

1. Prompt registries defined in `registries.search` to be used for searching image.

1. Look for sigstore configuration in `/etc/containers/registries.d`.

   1. Use "default-docker" configuration

   1. Fallback to `~/.local/share/containers/sigstore`

1. Look for certs and private keys in `/etc/docker/certs.d/{registry url}`

1. Look for v2 HTTPS / HTTP endpoints that return `401`

1. Download image to `~/.local/share/containers/`

1. Shortname cache created at `~/.cache/containers/short-name-aliases.conf`

## Skopeo

```sh
# Inspect
skopeo inspect docker://docker.io/busybox:latest
skopeo inspect --tls-verify=false docker://localhost:5000/busybox:latest

# List Tags
skopeo list-tags docker://cr.io/node
skopeo list-tags --tls-verify=false docker://localhost:5000/busybox

# Copy from registry into `docker save` formatted file and re-tag
skopeo copy docker://busybox:latest docker-archive:/home/user/busybox.tar:registry:5000/busybox:latest

# Copy from registry to local directory
skopeo copy docker://docker.io/busybox:latest dir:/tmp/images

# Copy between registries
skopeo copy --dest-tls-verify=false \
  docker://docker.io/busybox:latest docker://localhost:5000/busybox:latest

# Copy between registries with encryption and signing
skopeo copy \
  --encryption-key pgp:your_email@address.com \
  --sign-by your_email@address.com \
  --dest-tls-verify=false \
  docker://docker.io/busybox:latest docker://localhost:5000/busybox:latest

# Copy from registry to local directory in `OCI Layout Specification`
skopeo copy \
  --src-tls-verify=false \
  docker://localhost:5000/busybox:latest oci:./busybox:encrypted

# Copy from between local directories.
skopeo copy oci:./busybox:encrypted dir:./busybox-decrypted

skopeo delete --tls-verify=false docker://localhost:5000/busybox:latest

# Sync repo
skopeo sync --src docker --dest dir cr.io/node:alpine /tmp/images
```

When using encrypted container images, you must provide the appropriate decryption keys to the container runtime. When using cri-o, this means placing the private keys under the `/etc/crio/keys` directory (with the default settings).

For OpenShift clusters, there are two challenges to providing decryption keys:

* The decryption keys must be provided to cri-o on every worker node.

* The keys should not be manually copied into worker node file systems.

## Windows Notes

* Use `--override-os=linux` flag to pull images for Linux OS

* Registry interactive login bug: Use `-u` and `-p` flag to login to registry

* Config files location

  ```bat
  # Location of /etc/containers (e.g. policy.json)
  <root dir>\etc\containers\

  # Location of `auth.json`
  %USERPROFILE%\.config\containers\auth.json
  ```

* Default signing policy file to create in `<root dir>\etc\containers\policy.json`

  ```json
  {
   "default": [{
       "type": "insecureAcceptAnything"
   }]
  }
  ```

## References

[Deploy Containers From Encrypted Images in Red Hat OpenShift on IBM Cloud Clusters](https://www.ibm.com/cloud/blog/deploy-containers-from-encrypted-images-in-red-hat-openshift-on-ibm-cloud-clusters)

[CRI-O Image Decryption Support](https://github.com/cri-o/cri-o/blob/main/tutorials/decryption.md)
