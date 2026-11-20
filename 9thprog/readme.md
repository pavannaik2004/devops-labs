minikube start # start the local cluster
minikube status # check it's running

cd 9thprog # folder with namespaces.yaml and pods.yaml

kubectl apply -f namespaces.yaml # create the dev and prod namespaces (do this first)
kubectl apply -f pods.yaml # create 4 pods: frontend + backend in dev and in prod

kubectl get namespaces # dev and prod should be listed

kubectl get pods -n dev # frontend-pod, backend-pod (Running)
kubectl get pods -n prod # frontend-pod, backend-pod (Running)

kubectl get pods # default namespace → "No resources found in default namespace."
kubectl get pods -A --show-labels # optional: all namespaces with their labels

kubectl get pods -n dev -l app=frontend # only frontend pods in dev
kubectl get pods -n prod -l app=backend # only backend pods in prod

kubectl delete pods --all # clean-up (as in the manual: only clears the default namespace)
kubectl delete svc --all # delete all services
kubectl delete deployments --all # delete all deployments

kubectl delete namespace dev prod # actually removes the dev/prod pods too
