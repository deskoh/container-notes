# Image Signing

## GPG (GNU Privacy Guard) Recipes

```sh
# List keys
gpg --list-keys

# Generate keys
gpg --gen-key

# Generate revocation key (to be kept separate from private key)
gpg --output ~/revocation.crt --gen-revoke your_email@address.com

# Export public key (to be provided to verifying party)
gpg --output /tmp/pub.gpg --armor --export your_email@address.com
```

## Configure Registries (sigtore required for verifying party)

```yml
# Root: /etc/containers/registries.d/default.yaml
default-docker:
  sigstore: http://localhost:8000
  sigstore-staging: file:///var/lib/containers/sigstore

# Non-root: ~/.config/containers/registries.d/default.yaml
default-docker:
  sigstore: http://localhost:8000
  sigstore-staging: file:///home/user/.local/share/containers/sigstore
```

## Sign and Push Images

```sh
GNUPGHOME=$HOME/.gnupg

# Using podman
podman push --tls-verify=false --sign-by your_email@address.com localhost:5000/alpine

# Using skopeo
skopeo copy --dest-tls-verify=false --sign-by your_email@address.com docker://docker.io/alpine:latest docker://localhost:5000/alpine:latest

# Signature in sigstore specified in `sigstore-staging` to be to be provided to verifying party
```

## Verifying Party: Start sigstore server

Input: Signature

```sh
# Root
sudo bash -c 'cd /var/lib/containers/sigstore && python3 -m http.server'

# Non-root
bash -c 'cd /home/user/.local/share/containers/sigstore && python3 -m http.server'
```

## Verifying Party: Enforce Signature Verification

Input: GPG public key

```jsonc
// Root: /etc/containers/policy.json
// Non-root: ~/.config/containers/policy.json
{
  "default": [{ "type": "insecureAcceptAnything" }],
  "transports": {
    "docker": {
      "localhost:5000": [
        {
          "type": "signedBy",
          "keyType": "GPGKeys",
          "keyPath": "/tmp/key.gpg"
        }
      ]
    }
  }
}
```
