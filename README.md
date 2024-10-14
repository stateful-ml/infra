## setup

1. Create the cluster
```bash
minikube start --insecure-registry "192.168.39.0/24"
```
The insecure registries added are literally all possible expected minikube ips
2. enable registry as per the [docs](https://minikube.sigs.k8s.io/docs/handbook/pushing/#4-pushing-to-an-in-cluster-using-registry-addon)

```bash
minikube addons enable registry
minikube ip
```
add the minikube ip to docker insecure registries, e.g for snap it's `code /var/snap/docker/current/config/daemon.json`, adding `"insecure-registries" : [ "<minikube ip>:5000" ]` and restarting with `sudo snap restart docker`

3. Deploy the infra
```bash
helmfile sync
kubectl apply -f kube
```
### The steps below assume you want to deploy pipelines from the host. (It felt easier to set up in the beginnig and then sunk cost fallacy won)
4. (If you wanna deploy pipelines from the host) Port-forward the prefect-server to host
```bash
kubectl port-forward svc/prefect-server 4200:4200 &
```
... or press the button in Lens. Note that the UI will hit localhost:4200 cus it dont care, so you dont have the option of mapping this to any other port

5. Set up env in stateful-ml and go ham

6. Dont forget access to in-cluster stuff is done through a hacky throwaway pod (because its easier to rebuild the image on the fly with make)
