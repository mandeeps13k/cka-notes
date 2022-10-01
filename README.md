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



