## Config Maps & Secrets
* A ConfigMap allows us to define application related data.
* Now lets create a pod which mounts the config map [Refer Here](https://github.com/DevopsEasy/Kubernetes/tree/main/Deployment/ConfigMap) for the changes and create the pod
* Config maps can be mounted to the pods as volumes as well

```
kubectl apply -f configmap.yml
kubectl apply -f configpod.yml
kubectl exec -it <pod_name> -- /bin/sh
## execute commands
set, df -h 
ls /etc/appconfig/
cat /etc/appconfig/name1
```
* Secret is also much like config map but in secrets the values are base64 encoded
* Kubernetes secretes has 3 available commands
   * generic: generic secret holds any key value pair
   * tls: secret for holding private-public key for communicating with TLS protocol
   * docker-registry: This is special kind of secret that stores usernames and passwords to connect to private registries
* Create a secret

```
ubectl create secret generic first-secret --from-literal "password=india@123"
kubectl get secrets
kubectl describe secrets first-secret 
kubectl exec -it secret-demo -- /bin/sh 
```
* Like configmap secrets also can be mounted as a volume

## Kuberenets as a Service on Cloud Platforms
* Cloud providers like AWS, Azure , Google offer kubernetes as a service
* When we use these offerings
   * Google Kuberenetes Engine
   * Azure Kubernetes Services
   * AWS Elastic Kubernetes Services
* The cloud provider will manage
   * the k8s master nodes
   * the networking configuration
   * Load balancing and ingress capabilities
   * Persistent Volume native support to the clouds block and file storage
   * Integrated logging and monitoring support

## EKS Cluster Setup
* [Refer Here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) for official docs.
* Install kubectl, eksctl, configure iam user.
* To create a cluster execute the below command
```
eksctl create cluster \ 
--name my-cluster \
--region us-west-2 \
--with-oidc \
--ssh-access \
--ssh-public-key k8s \
--managed
```
* To delete a cluster
```
eksctl get cluster
ekssctl delete cluster <cluster_name>
```
