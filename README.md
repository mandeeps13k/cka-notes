# kubectl-commands


## create/apply

kubectl apply -f pod1.yml \
kubectl create -f pod-definition.yml

## get details
kubectl get pods \
kubectl get nodes \
kubectl get services \
kubectl get deployments \
kubectl describe pod pod-name 
## delete/destroy
kubectl delete pod pod-name\
kubectl delete -f rc-definition.yaml
## replicas/scale
kubectl get replicaset\
kubectl get rs\
kubectl describe replicaset myapp-replicaset\
kubectl replace -f replicaset-definition.yaml\
kubectl scale --replicas=6 -f replicaset-definition.yaml\
kubectl scale replicaset new-replica-set --replicas=5\
kubectl delete replicaset myapp-replicaset\
kubectl scale deployment.apps/webapp --replicas=3
## Deployments
kubectl create -f deployment-definition.yaml\
kubectl get deployments\
kubectl delete deployment myapp-deployment\
kubectl describe deployments myapp-deployment-1\
kubectl get all
## Dry-Run
kubectl run nginx --image=nginx --labels="app=myapp" --dry-run=client -o yaml > nginx-app.yaml\
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
## NameSpaces

## Imperative Commands
kubectl run --image=nginx nginx\
kubectl create deployment --image=nginx nginx\
kubectl expose deployment nginx --port 80\
kubectl edit deployment nginx\
kubectl scale deployment nginx --replicas=5\
kubectl set image deployment nginx nginx=nginx:1.18\
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml\
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml\
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml

## Create a Redis Pod and a Service to expose it
kubectl run redis --image=redis:alpine --dry-run=client -o yaml > redis-pod.yaml\
kubectl expose pod redis --port=6379 --name=redis-service

## Create Pod and expose on a Port
kubectl run custom-nginx --image=nginx --port=8080\
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns\

## Labels , Selectors
kubectl get pods --selector env=dev\
kubectl get pods --selector bu=finance --no-headers\
kubectl get all --selector env=prod\
kubectl get pod --selector env=prod,bu=finance,tier=frontend
## Taints, Tolerations
kubectl taint nodes node1 app=blue:NoSchedule\
kubectl describe nodes controlplane | grep Taint\
kubectl taint nodes node1 app=blue:NoSchedule- (to remove a taint)
## Label Node
kubectl label nodes node01 size=Large\
kubectl label node node01 color=blue\
## Static Pods - Static Pod config files are placed in /etc/kubernetes/manifests ; kubernetes controlplane components are deployed as static pods
kubectl run --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml\
kubelet path - /etc/kubernetes/manifests ; /var/lib/kubelet/config.yaml\
kubectl create configmap -n kube-system my-scheduler-config --from-file=/root/my-scheduler-config.yaml 
## Monitoring & Logs
kubectl logs -f pod-name container-name\
kubectl top pod\
kubectl top node\
k logs deployments/collect-data -c nginx (container nginx only logs)
## Rollout & Rollback
kubectl rollout status deployment/myapp-deployment\
kubectl rollout history deployment/myapp-deployment\
kubectl rollout undo deployment/myapp-deployment (RollBack)
## Commands & Arguments , Secrets and ConfigMaps
Entrypoint (in Dockerfile) -> Command (in Pod YAML) ; \
CMD (in Dockerfile) -> args (in Pod YAML); CMD Gets appended to Entrypoint \
kubectl create secret generic app-secret --from-literal=key=value \
kubectl exec shell-demo -- ps aux
## Cluster Maintainence
kubectl get all -o wide \
kubectl cordon node01 \
kubectl drain node01 
## BackUp , ETCD
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml \
etcdctl snapshot save snapshot.db --cacert --cert --endpoint --key \
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db \

ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db \

To restore backup, update volumeMounts in /etc/kubernetes/manifests/etcd.yaml ; /var/lib/etcd -> /var/lib/etcd-from-backup \
Always check --data-dir to verify etcd data directory \
etcd-server ~ ➜  ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new \
etcd service config -> vi /etc/systemd/system/etcd.service \
etcd-server ~ ➜  systemctl daemon-reload \
etcd-server ~ ➜  systemctl restart etcd
## Certificates
kubectl get csr \
kubectl certificate approve NAME \
kubectl certificate deny NAME \
kubectl delete csr NAME
## Config, Roles
kubectl config view --kubeconfig my-kube-config \
kubectl config get-clusters --kubeconfig /root/my-kube-config \
kubectl config get-contexts --kubeconfig /root/my-kube-config \
kubectl config use-context --kubeconfig=/root/my-kube-config research \
kubectl config view --kubeconfig=/root/my-kube-config \
kubectl config --kubeconfig=/root/my-kube-config current-context \
kubectl get roles \
kubectl get rolebinding \
kubectl auth can-i get pods --as dev-user \
kubectl get pods --as dev-user 
## Networking
Check kubernetes network plugin -> ps -aux | grep kubelet | grep network \
CNI Plugin Path -> ls /opt/cni/bin \
CNI Plugin configured on kubernetes cluster -> /etc/cni/net.d/ \
For interface -> ip link ; ip addr; ip route (execute on node )  ; For ip -> also check kubectl logs pod \
k get nodes -o wide ; ipcalc -b 10.51.11.3/24 \
Services IP Defined in /etc/kubernetes/manifests/kube-apiserver.yaml (--service-cluster-ip-range=10.96.0.0/12) \
kube-proxy Proxy type -> kubectl logs kube-proxy-4ttzb -n kube-system ; kube-proxy runs as daemonsets

## Network Policies

The namespaceSelector from NPs works with Namespace labels ; \
Use nameSpaceSelector (also as kubernetes used Automatic Labelling , The Kubernetes control plane sets an immutable label kubernetes.io/metadata.name on all namespaces) \

k get ns --show-labels

## Troubleshooting

Deployment Logs -> \
k logs deployments/collect-data -c nginx (container nginx only logs)
If multiple container try to run on same port in a Pod, first will success, other will fail, check logs. 

Kubectl Broken -> \
check kube-apiserver Pod logs -> /var/log/pods/kube-system_kube-apiserver-controlplane_a94588ab9b1141fe60c1fb951b60579a/kube-apiserver ; \
check running pods -> crictl ps ; \
crictl logs container-id ; \
or docker ps ; docker logs ; (when docker is configured)
for kube-apiserver -> logs -> also check -> journalctl | grep apiserver ; \
tail -f /var/log/syslog | grep apiserver ; /var/log/pods ; crictl logs


Kubelet -> \
kubelet config on nodes -> check kubelet pod or check kubelet service on node -> service kubelet status ; cat /var/lib/kubelet/config.yaml \
vi /etc/kubernetes/kubelet.conf \
journalctl -u kubelet \
journalctl -u kubelet | grep failed \
journalctl -u kubelet | grep error \
kubelet logs: /var/log/syslog or journalctl , grep for apiserver or error for debugging

kube-proxy -> \
kube-proxy config as daemon set ; kubectl get ds -n kube-system; kube-proxy run kube-proxyconfig-map ; \
check kubectl get cm -n kube-system ; kube-proxy runs in kube-system -> check kubectl get ds -n kube-system ;  \
check /var/lib/kube-proxy/kubeconfig.conf for kube-proxy config ;  

Control-Plane Failure -> \
check running pods in -n kube-system namespace ; \
check kube-scheduler static pod config in /etc/kubernetes/manifests/kube-scheduler.yaml 

Kube-controller-manager -> \
check running static pods in -n kube-system namespace ; \
cgeck kube-controller-manager static pod config in /etc/kubernetes/manifests/kube-controller-manager.yaml

Other errors-> \
For other errors related to file or Directory Not found , also check volume Mounts , hostPath on the static Pods as the certificates are mounted as volume
check journalctl | grep apiserver \
cat /var/log/syslog | grep kube-apiserver ;

## Ingress

For Ingress creation and related Issues, always check Ingress Controller (service) for ports config \
Also check ingress class with k get ingressclass ; \
k create ingress --help ;


## Important Points to Remember 

- When creating Ingress, always check Ingress Class as `ingressClassName` is required while creating Ingress, also check Ingress Controller
- When performing backUp and restore of ETCD, make sure the volumeMounts and Paths are correctly updated in the ETCD Static Pod Config `/etc/kubernetes/manifests/etcd.yaml` . 
- For an external ETCD Configuration , where ETCD is running as a process on an external etcd-server, perform an ETCD Restore on the external ETCD Server by ssh into the external ETCD Server. Inspect the local etcd process by running `ps -ef | grep etcd`. When performing a restore on the external ETCD Server, make sure to update the `--data-dir` of backup restore to the local etcd process in `/etc/systemd/system/etcd.service`. Also make sure the `--data-dir` permissions are correct by running `chown -R etcd:etcd /var/lib/etcd-data-new` & restart ETCD local process after updating - `systemctl daemon-reload` & `systemctl restart etcd`
- Remember that `--data-dir` is the Path where the snapshot will be ReStored, and this path should be updated in /etc/kubernetes/manifests/etcd.yaml , in the command path as well as the `volumeMount` path.
- Use `k expose deploy` command to create service for a deployment - it applies right selectors automatically.
- Always check selectors in services
- The Storage Class that makes use of VolumeBindingMode set to WaitForFirstConsumer. This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.
- While creating network policy, only specify `ingress` or `egress` sections when making changes. Do not use empty array [ ] in these sections.















