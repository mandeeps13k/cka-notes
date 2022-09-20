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
kubectl describe replicaset myapp-replicaset\
kubectl replace -f replicaset-definition.yaml\
kubectl scale --replicas=6 -f replicaset-definition.yaml\
kubectl delete replicaset myapp-replicaset

