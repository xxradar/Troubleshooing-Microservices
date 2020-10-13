# Troubleshooting Microservices on K8S and Calico

## Get to know your cluster
### Internode connectivity
```
sudo calicoctl node status
```
### IPPools settings (NatOutgoing, Encapsulation, pool cidr and block 
```
calicoctl get ippools

ex. calicoctl get ippools default-ipv4-ippool -o yaml
```
### Verify the IP address assignment
```
calicoctl ipam show --show-blocks
```
### Verify pool annotations
Annotate a namesmace to use an specific ippool (should already be done)
```
kubectl annotate namespace wwwdemo "cni.projectcalico.org/ipv4pools"='["demo-ipip-ippool"]'
```
or/and to verify
```
kubectl get ns wwwdemo -o yaml
```
## Check application connectivity  
### Verify the IP address of a pod
```
calicoctl get workloadEndpoint --all-namespaces -o wide #You can find the cali network interface easily

kubectl get po -o wide -A
```


