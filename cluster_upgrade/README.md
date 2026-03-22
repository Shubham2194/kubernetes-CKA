#Cluster Upgrade

Prerequisites:
- Cordon nodes(Node unschdeulables - stop new deployments)
- Release Notes(change log from 1.31 > 1.32)
- Test in lower env
- Cotrol plane version = Node version
- Five available IP address within Subnet
- kubelet version = control plane version

Process to Upgrade:
- Upgrade Control plane
- Upgrade node group/nodes/Fargate
- Upgrade Addons(kube-proxy , VPC-cni , coredns)


