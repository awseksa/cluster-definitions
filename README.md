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
 <p align="center">< ![image](https://github.com/user-attachments/assets/aaa254da-f030-43b8-8e1b-2946a79aed1e) align="center"></p>
![image](https://github.com/user-attachments/assets/aaa254da-f030-43b8-8e1b-2946a79aed1e)

  
