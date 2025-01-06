## setup

1. Create the cluster
insecure-registry is here for the builtin docker registry over http (handy)
cpus are 6 because the default of 2 is not enough duh
```bash
minikube start --insecure-registry "192.168.39.0/24" --cpus 6
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

7. When the time comes to use github actions to work with the cluster, use the hacky ngrok solution to avoid paying for infra :)
- 7.1 get a free ngrok domain
- 7.2 set up the ngrok config and run it with `ngrok start --all --config ngrok-config.yaml`, use  .ngrok-config.example for guidance
- 7.3 setup helm gh actions, use .kubekonfig.example file to set up the KUBECONFIG_B64 secret and `kubectl create token ci` command to set up the user token inside

8. Create some namespaces depending on deployment-manager setup, with current stup youll need `kubectl create namespace stg` and `kubectl create namespace prd`

## Why hacky ngrok agent?
Yes, ngrok ingress exists and so does ngrok gateway. In theory, one could use them instead of the command line agent with this reliance on nodeports, but:
- mlflow does not respect path rewriting (afaik) during its miriad of redirects so ingress is out of the question
- ngrok gateway controller hardcodes port 433 so using routes on different ports for different services is out of the question


## TODO:
- add real ingresses to ditch this whole shaman dance with ngrok + nodeport + port forwarding
