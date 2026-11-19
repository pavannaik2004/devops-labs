cd ~ ; pwd ; ls # move to the home folder

kubectl create deployment nginx-deployment
--image=nginx:latest
--replicas=2
--port=80
--dry-run=client -o yaml > nginx-deployment.yaml

kubectl create service nodeport nginx-service
--tcp=80:80
--node-port=30080
--dry-run=client -o yaml > nginx-service.yaml

nano nginx-service.yaml # look at the generated files
nano nginx-deployment.yaml

minikube start # start the local cluster
minikube status # check it's running

minikube delete
minikube start --driver=docker

kubectl get deployments # run first to show nothing exists yet
kubectl get replicasets # → "No resources found"
kubectl get pods

kubectl apply -f nginx-deployment.yaml # create the Deployment from the file

kubectl get deployments # nginx-deployment 2/2 ready
kubectl get replicasets # the ReplicaSet that keeps 2 pods running
kubectl get pods # 2 nginx pods, both Running

kubectl expose deployment nginx-deployment
--type=NodePort -
-port=80

kubectl delete service nginx-deployment # remove the old service
kubectl expose deployment nginx-deployment --type=NodePort --port=80 # re-run, now "exposed"

kubectl delete svc --all # delete all services
kubectl delete deployments --all # delete all deployments (and their pods)
