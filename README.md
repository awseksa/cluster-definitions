Flux installation using Kustomization on workload clusters

Create two git main repositories to store management and workload clusters.
1.	Cluster definitions repo contains management cluster, flux configuration and workload cluster configuration files.
•	Create a folder with your workload cluster name and upload the cluster configuration file and kustomization file.
```bash
cluster-definitions
├── clusters
│   └──hpe-gitops-mgmt
│      ├── flux-system
│      │   ├── gotk-components.yaml
│      │   ├── gotk-sync.yaml
│      │   └── kustomization.yaml
│      │   └──.sops.pub.asc
│      ├── hpe-gitops-mgmt
│      │   └── eksa-system
│      │       ├── eksa-cluster.yaml
│      │       └── kustomization.yaml
│      └── dell-gitops-workload
│          ├── eksa-cluster.yaml
│          └── kustombootstrap.yaml
└── README.md
```
2.	Cluster components repo contains application deployment and flux configuration files created and uploaded manually.
```bash
cluster-components
├── components
│   ├── clusters
│   │   └──dell-gitops-worload
│   │      └── flux-system
│   │          ├── gotk-components.yaml
│   │          └── kustomization.yaml
│   └── nginx
│       └── nginx.yaml     
└── README.md
```

Workflow architecture:


![image](https://github.com/user-attachments/assets/aaa254da-f030-43b8-8e1b-2946a79aed1e)

  
Integration of SOPS on the clusters to encrypt and decrypt the secrets automatically:
Prerequisites:
1.	Install SOPS:
# Download the binary
curl -LO https://github.com/getsops/sops/releases/download/v3.9.1/sops-v3.9.1.linux.amd64
# Move the binary in to your PATH
mv sops-v3.9.1. linux.amd64 /usr/local/bin/sops
# Make the binary executable
chmod +x /usr/local/bin/sops
2.	Generate a GPG key:
gpg --batch --full-generate-key <<EOF
%no-protection
Key-Type: 1
Key-Length: 4096
Subkey-Type: 1
Subkey-Length: 4096
Expire-Date: 0
Name-Comment: #using to encrypt/decrypt secrets
Name-Real: awseksa
EOF

Output:
gpg: key 55DFBD651E0E85AD marked as ultimately trusted
gpg: revocation certificate stored as '/root/.gnupg/openpgp-revocs.d/A96F159548F6F5A2A3663C6055DFBD651E0E85AD.rev'

3.	Verify the key generate using the command:
gpg --list-secret-keys awseksa

output:
root@awseksa:/eksa-clusters# gpg --list-secret-keys awseksa
sec   rsa4096 2024-11-18 [SCEA]
      A96F159548F6F5A2A3663C6055DFBD651E0E85AD
uid           [ultimate] awseksa (#using to encrypt/decrypt secrets)
ssb   rsa4096 2024-11-18 [SEA]

4.	Create a secret in the management cluster using the above generated key:
gpg --export-secret-keys --armor A96F159548F6F5A2A3663C6055DFBD651E0E85AD |
kubectl create secret generic sops-gpg \
--namespace=flux-system \
--from-file=sops.asc=/dev/stdin

Output:
secret/sops-gpg created

5.	Create a public key using the generated key and upload public key to the management cluster github path (clusters/hpe-gitops-mgmt/flux-system):
gpg --export --armor A96F159548F6F5A2A3663C6055DFBD651E0E85AD >.sops.pub.asc

6.	Go to the github path clusters/hpe-gitops-mgmt/flux-system and edit the gotk-sync.yaml and add the below content:
  decryption:
    provider: sops
    secretRef:
      name: sops-gpg

7.	Now encrypt your secret using the generated key:
Encrypt the secret with SOPS using your GPG key:
sops --encrypt --pgp A96F159548F6F5A2A3663C6055DFBD651E0E85AD --encrypted-regex '^(data|stringData)$' test.yaml > testenc.yaml

output:

apiVersion: v1
kind: Secret
metadata:
    name: github
    namespace: flux-system
type: Opaque
stringData:
    username: ENC[AES256_GCM,data:o5++Bb91WA==,iv:tyOTL+tpkmq4It7f5C+PtlCOX/7fych69i9TtuBOnss=,tag:FuqBECPU4i/GMh3d1iLxMw==,type:str]
    password: ENC[AES256_GCM,data:ngK91/QVB+wnHXWrYfaNsTcepvqzH9+kniKQLtbSZJXVHZeiJnaoXA==,iv:K7DcH/thgnKgIXZex+URSxqxaRs0q+jV1QGfLNIA/nI=,tag:Z5g22X7Clvk65vFiN2kwRQ==,type:str]
sops:
    kms: []
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age: []
    lastmodified: "2024-11-18T06:40:53Z"
    mac: ENC[AES256_GCM,data:n5dwrIfUjxq/Bc5lFJ0DD8usRHT1joAOfEyiEdxQCN+HXPRxpEwvAaq7i0ssO3ubTDRDRXReUlGYFXmDNsGadxk/uRg+m8pBnRKqcBU30Zks/jay4dsiGd7SozH2iGdrDRLuFEEykfYSu/1Cc0fXqDkSG9v0sMfe
4GPPiwltAp0=,iv:l9qlkUYV0JB8TAeC5+DQMlpguEIzCdYh5YcnbesR0AE=,tag:eVXobCeU/ciWZ+I9f7/+qQ==,type:str]
    pgp:
        - created_at: "2024-11-18T06:40:53Z"
          enc: |-
            -----BEGIN PGP MESSAGE-----

            hQIMAw9uClCp5jyCAQ//c2Uoipsi6aq/TFLwOxCVL5CFMruMZ6T0luEyJSKnOevC
            loT3OQkOcUVwH9s6bJ7WxHQDTZCpP7bI+4SebMkJTx2cGQ+I2xRON73jzozKj/P/
            uWFu913PyMz0e3ECRBP62RZOlSMBRtizUzEcUacjrQqrOJW1/8VZFonEi0EVdKX9
            VRpE4YYazWkY5ma8Wlv/6JChvNFP0ZDjIXp3VUqgTZqtpBQlQORg/DNOyTtkmiIW
            TULEea7xTKT/pCTL2oHrif+gbwc3nYjP37aLzrAbmC+S4r1pDz6fmIZtghJ16zTg
            hDfFpF2ZNEPszKtiQA0zO6r2Ts8FNWObqS71sCCEAqidce9kLMmSlNgPUZpGfWhb
            YPG5pN37g1FFVR0n0cDrpVpOqYTQANtMRCgL3NOPZB8deME1WWnu2HLdMYzuM9n7
            z8TW4Oxe6UAlY5yDopMxgTbyx+rzb/sKzlyvXTJqGfRkP0qUF8w6ebeXk8aw4Yrs
            dTso+pqIPz9qmw4BHds+EysBBBkxITBs2JGNRnVXpvbopOtnowFD+jKB9mmN5xcA
            +EWsBlQWT1F54xQp24ubX5Ui3pOmFlhwH348tpDwLU8AdueWxJ5gmWeOztwUj0u9
            CoDgYshYa3VMZaM93cIKe1sNgoZUfXq0kA+7qtAhUMyrIbVkVws4k37aFiWec3DS
            XgHhc/m+nlBJOKy/AeFEOEYGYndFOBDqMicudyN0qux5oGcr9R9hJqOE4wQVkFdD
            iyr/rMLMcZjrje1L3SvimBeVVoVUtJCE8VzoDigYZ4C83slg2T4+CcQNAng8uME=
            =NQdE
            -----END PGP MESSAGE-----
          fp: A96F159548F6F5A2A3663C6055DFBD651E0E85AD
    encrypted_regex: ^(data|stringData)$
    version: 3.9.1


8.	Configure and upload the kustomization bootstrap yaml file to github path clusters/hpe-gitops-mgmt/dell-gitops-workload and verify the successful deployment:
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: git-components
  namespace: flux-system
spec:
  interval: 5m0s
  url: https://github.com/awseksa/cluster-components
  ref:
    branch: main
  secretRef:
    name: github
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: deploy-application
  namespace: eksa-system
spec:
  interval: 5m
  path: "./components/clusters/dell-gitops-workload/"
  prune: true
  sourceRef:
    kind: GitRepository
    name: git-components
    namespace: flux-system
  kubeConfig:
    secretRef:
      name: dell-gitops-workload-kubeconfig
---
apiVersion: v1
kind: Secret
metadata:
    name: github
    namespace: flux-system
type: Opaque
stringData:
    username: ENC[AES256_GCM,data:o5++Bb91WA==,iv:tyOTL+tpkmq4It7f5C+PtlCOX/7fych69i9TtuBOnss=,tag:FuqBECPU4i/GMh3d1iLxMw==,type:str]
    password: ENC[AES256_GCM,data:ngK91/QVB+wnHXWrYfaNsTcepvqzH9+kniKQLtbSZJXVHZeiJnaoXA==,iv:K7DcH/thgnKgIXZex+URSxqxaRs0q+jV1QGfLNIA/nI=,tag:Z5g22X7Clvk65vFiN2kwRQ==,type:str]
sops:
    kms: []
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age: []
    lastmodified: "2024-11-18T06:40:53Z"
    mac: ENC[AES256_GCM,data:n5dwrIfUjxq/Bc5lFJ0DD8usRHT1joAOfEyiEdxQCN+HXPRxpEwvAaq7i0ssO3ubTDRDRXReUlGYFXmDNsGadxk/uRg+m8pBnRKqcBU30Zks/jay4dsiGd7SozH2iGdrDRLuFEEykfYSu/1Cc0fXqDkSG9v0sMfe4GPPiwltAp0=,iv:l9qlkUYV0JB8TAeC5+DQMlpguEIzCdYh5YcnbesR0AE=,tag:eVXobCeU/ciWZ+I9f7/+qQ==,type:str]
    pgp:
        - created_at: "2024-11-18T06:40:53Z"
          enc: |-
            -----BEGIN PGP MESSAGE-----

            hQIMAw9uClCp5jyCAQ//c2Uoipsi6aq/TFLwOxCVL5CFMruMZ6T0luEyJSKnOevC
            loT3OQkOcUVwH9s6bJ7WxHQDTZCpP7bI+4SebMkJTx2cGQ+I2xRON73jzozKj/P/
            uWFu913PyMz0e3ECRBP62RZOlSMBRtizUzEcUacjrQqrOJW1/8VZFonEi0EVdKX9
            VRpE4YYazWkY5ma8Wlv/6JChvNFP0ZDjIXp3VUqgTZqtpBQlQORg/DNOyTtkmiIW
            TULEea7xTKT/pCTL2oHrif+gbwc3nYjP37aLzrAbmC+S4r1pDz6fmIZtghJ16zTg
            hDfFpF2ZNEPszKtiQA0zO6r2Ts8FNWObqS71sCCEAqidce9kLMmSlNgPUZpGfWhb
            YPG5pN37g1FFVR0n0cDrpVpOqYTQANtMRCgL3NOPZB8deME1WWnu2HLdMYzuM9n7
            z8TW4Oxe6UAlY5yDopMxgTbyx+rzb/sKzlyvXTJqGfRkP0qUF8w6ebeXk8aw4Yrs
            dTso+pqIPz9qmw4BHds+EysBBBkxITBs2JGNRnVXpvbopOtnowFD+jKB9mmN5xcA
            +EWsBlQWT1F54xQp24ubX5Ui3pOmFlhwH348tpDwLU8AdueWxJ5gmWeOztwUj0u9
            CoDgYshYa3VMZaM93cIKe1sNgoZUfXq0kA+7qtAhUMyrIbVkVws4k37aFiWec3DS
            XgHhc/m+nlBJOKy/AeFEOEYGYndFOBDqMicudyN0qux5oGcr9R9hJqOE4wQVkFdD
            iyr/rMLMcZjrje1L3SvimBeVVoVUtJCE8VzoDigYZ4C83slg2T4+CcQNAng8uME=
            =NQdE
            -----END PGP MESSAGE-----
          fp: A96F159548F6F5A2A3663C6055DFBD651E0E85AD
    encrypted_regex: ^(data|stringData)$
    version: 3.9.1
