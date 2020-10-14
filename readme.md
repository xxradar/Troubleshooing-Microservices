# Troubleshooting Microservices on K8S and Calico
The examples are based on app-routable-demo https://github.com/xxradar/app_routable_demo.

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
### Tcpdump via kubectl patch
```
vi patch.yaml

spec:
  template:
    spec:
      containers:
      - name: tcpdumper
        image: docker.io/dockersec/tcpdump
```
```
kubectl patch deployment echoserver-1-deployment -n app-routable-demo --patch "$(cat patch.yaml)"
```
``` 
kubectl get pods -n app-routable-demo  #choose a pod
ex. kubectl attach -n app-routable-demo echoserver-1-deployment-598f4c696b-44mwg
```
To undo the changes
```
kubectl rollout undo deployment/echoserver-1-deployment -n app-routable-demo
```

### Tcpdump hostNetwork
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug
spec:
  hostNetwork: true
  containers:
  - name: debug
    image: docker.io/xxradar/hackon
    command: ["bash"]
    args: ["-c", "sleep 100"]
  nodeSelector:
    kubernetes.io/hostname: ip-10-11-2-123
EOF
```
```
kubectl exec -it debug -- bash
```
## Debug Network Policies
### Find the podSelector the network policy applies to 
```
kubectl get netpol access-zone1  -n app-routable-demo -o yaml #Find the podSelector 
```
### Find the pods the podSelector applies to
```
kubectl get po -l app=nginx-zone1 -n app-routable-demo
```
### Find the name of the policies the labels applies to 
```
kubectl get netpol -A -o json | jq -r '.items[] | select(.spec.podSelector.matchLabels.app == "echoserver-1") | .metadata.name'
```

