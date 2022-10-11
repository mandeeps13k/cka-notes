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
## Troubleshooting
Kubelet -> \
kubelet config on nodes -> check kubelet pod or check kubelet service on node -> service kubelet status ; cat /var/lib/kubelet/config.yaml \
vi /etc/kubernetes/kubelet.conf \
journalctl -u kubelet \
journalctl -u kubelet | grep failed \
journalctl -u kubelet | grep error

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

For other errors related to file or Directory Not found , also check volume Mounts , hostPath on the static Pods as the certificates are mounted as volume














