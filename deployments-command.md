Kubernetes Common Commands

#Kubectl apply all files in a directory
kubectl apply --recursive --filename

#Delete all Kubernetes objects recursively in a directory
kubectl delete --recursive --filename

kubectl delete -R -f

#Port-Forwarding to map to local terminal
kubectl port-forward deployment/<DeploymentName> <port>:<port>

#Describe deployment to find details of deployment
kubectl describe deploy <DeploymentName>

#Get events related to deployment
kubectl get events --field-selector involvedObject.name=<DeploymentName>


#Visualize your Kubernetes workloads
Octant is an open source developer-centric web interface for Kubernetes that lets you inspect a Kubernetes cluster and its applications.
https://octant.dev/

#kubectl plugins manager
krew 
