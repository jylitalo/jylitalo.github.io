---
title: "SOPS for Secrets in GitOps"
date: 2026-11-17T12:00:00+02:00
categories: ["index", "homelab", "linux", "k3s", "secret"]
---
Secrets are something that you usually need in one form or another when you deploy services into kubernetes cluster.
One can easily add them with `kubectl apply ...` to your kubernetes, but you don't want to commit secrets as plain text
into git repositories, which is a problem if you want to use GitOps to manage cluster.

Kubernetes has number of options that you can use to keep your secrets sealed secrets, external secrets, ...
but SOPS with age is probably the easiest one to encrypt them in such way that you feel confident to save them
into git repository.
<!--more-->
## Requirements

### Software
SOPS (Secrets OPerationS) is an open-source file encryption and secret management tool created by Mozilla and
currently hosted as a project by the Cloud Native Computing Foundation (CNCF). With this kind of background, one might
assume that you just `dnf install sops` in Fedora or `apt install sops` in Debian, but it is not that simple,
because those distributions don't have sops as official package.

Instead of installing sops as package, I decided to solve the problem with container. I installed
`podman` and then commandde `podman pull ghcr.io/getsops/sops:v3.13.3`. You will also need encryption tool called `age`
(which is available in Fedora and Debian) to generate our public-private keypair for SOPS. On this use case, you nee
`age` only for keygeneration and once you've done it, you can remove it.

### Public-private keypair

Commands that create directory and actual keypair are:
```
mkdir -p ~/.config/sops/age
chmod 700 ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt
chmod 600 ~/.config/sops/age/keys.txt
```

`age-keygen` command will print something like `Public key: age18g5wumcktr4tyzdtdtcayet8heuyqhwwg7dx3a85xs58pjxptu5qw6edlg`
That `age18g5wumcktr4tyzdtdtcayet8heuyqhwwg7dx3a85xs58pjxptu5qw6edlg`, which is public key part of your keypair.
Here is template for `.sops.yaml` that needs to be stored into your gitops repository (replace `age1...` with your public key):
```
creation_rules:
  - path_regex: .*\.sops\.ya?ml$
    encrypted_regex: ^(data|stringData)$
    age: age18g5wumcktr4tyzdtdtcayet8heuyqhwwg7dx3a85xs58pjxptu5qw6edlg
```

`~/.config/sops/age/keys.txt` is the private key of your keypair, which you should NOT commit into git repository.
Instead of `git`, we will use kubectl to add it into `kubectl -n flux-system create secret generic sops-age --from-file=identity.agekey=$HOME/.config/sops/age/keys.txt` 

### Tell FluxCD about SOPS

Add following block to your `gotk-sync.yml` file:
```
spec:
  decryption:
    provider: sops
    secretRef:
      name: sops-age
```

Note that the `name` in `secretRef` will have to match with what you stored on previous section with `kubectl -n flux-system create secret ...`

## Test Encrypt and Decrypt

Create test-secret.yaml in root of your gitops repo:
```
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
  namespace: default
stringData:
  username: testuser
  password: super-secret-password
```

Encrypt that file with your keypair:
```
 podman run --rm \ 
  -v "$PWD:/work:Z" \
  -v "$HOME/.config/sops/age:/age:ro,Z" \
  -e SOPS_AGE_KEY_FILE=/age/keys.txt \
  -w /work \
  ghcr.io/getsops/sops:v3.13.3 \
  --encrypt \
  --output /work/encrypted-test-secret.sops.yaml \
  /work/test-secret.sops.yaml
```

and to decrypt it:
```
 podman run --rm \ 
  -v "$PWD:/work:Z" \
  -v "$HOME/.config/sops/age:/age:ro,Z" \
  -e SOPS_AGE_KEY_FILE=/age/keys.txt \
  -w /work \
  ghcr.io/getsops/sops:v3.13.3 \
  --encrypt \
  --output /work/decrypted-test-secret.sops.yaml \
  /work/encrypted-test-secret.sops.yaml
```

`decrypted-test-secret.sops.yaml` should be identical with `test-secret.yaml` file.

## Migration from `kubectl apply` secrets to SOPS secrets via fluxcd

If your kubernetes cluster has secrets, which have been created with `kubectl` and are not tracked
with GitOps, you might want to migrate those secrets in encrypted format into your gitops repository.

Transition needs to go through following steps:
1. get current secret from kubernetes `kubectl get secret test-secret -o yaml` to file
2. drop `annotations`, `creationTimestamp`, `resourceVersion` and `uid` from `metadata`
3. encrypt the content (see previous section)
4. commit and push your encrypted secret into your git repository as `test-secret.sops.yaml` (`.sops.yaml` as suffix)
5. `flux reconcile kustomization flux-system --with-source`

With these changes, fluxcd will read encrypted version of secret from gitops repository, decrypt it
and add it into your kubernetes cluster in plain text, but this is still step to right direction
considering, that your git repository has encrypted version of those secrets.

`kubectl get secret test-secret -o yaml` should still show your secret in decrypted
format (without those additional `metadata` fields) even though the git repository it in encrypted version.
You can still use `SecretRef`s in your YAML manifests like you did before you encrypted the secrets with SOPS.

