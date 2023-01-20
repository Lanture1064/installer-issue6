Here is the steps about how to install bestchains BaaS platform

## 1. Install u4a-component
For the 1st step, we'll install u4a-component and it'll provide the account, authentication, authorization and audit funcationality built on Kubernetes. And it has the capability to add more features following the guide later.

And then we'll deploy BaaS on top of it, and use OIDC token for SSO between u4a and baas component.

### Install cluster tools
Before deploy u4a, we should add some tools for later usage. Enter into u4a-component folder and following the step below:

* This step will install a ingress nginx controller with ingressclass named 'u4a-component-ingress' and cert-manager for certificate management.

```
# 1. create a namespace to install u4a-component
$ kubectl create ns u4a-system

# 2. edit charts/cluster-component/values.yaml to replace '<replaced-ingress-node-name>'
# with the K8S node name that will install the ingress controller, so update the value of deployedHost, and remember the IP address of this host, will use it at the next step.

ingress-nginx:
  # MUST update this value
  deployedHost: &deployedHost
    k8s-ingress-nginx-node-name

# you should also update the image address if you're using a private registry, then you should replace 'hub.tenxcloud.com'(or the image name) with your private registry.

# 3. install cluster-component using 
$ helm install cluster-component -n u4a-system charts/cluster-component

# 4. check the status of pods to make sure ingress-nginx-controller and cert-manager are ready
$ kubectl get pods -n u4a-system
NAME                                                          READY   STATUS    RESTARTS   AGE
cert-manager-756fd78bff-wb2vh                                 1/1     Running   0          76m
cert-manager-cainjector-64685f8d48-qg69v                      1/1     Running   0          76m
cert-manager-webhook-5c46d68c6b-f4dkh                         1/1     Running   0          76m
cluster-component-ingress-nginx-controller-5bd67897dd-5m9n7   1/1     Running   0          76m
```
### Install u4a services
Enter into u4a-component folder and following the step below:

This step will install the following services:
* Capsule for tenant management
* kube-oidc-proxy for K8S OIDC enablement
* oidc-server for OIDC and iam service
* resource-view-controller for resource aggregation view from multiple clusters

1. Edit values.yaml to replace the placeholder below:
* `<replaced-ingress-nginx-ip>`, replace it with the IP address of the ingress nginx node that deployed in the previous step, this placeholder will have multiple ones
* `<replaced-oidc-proxy-node-name>`, replace it with the node name where kube-oidc-proxy will be installed
* `<replaced-k8s-ip-with-oidc-enabled>`, replace it with the IP address of node where kube-oidc-proxy will be installed, this placeholder will have multiple ones
* you should also update the image address if you're using a private registry

2. Install u4a component using helm
```
# run helm install
$ helm install --wait u4a-component -n u4a-system .

# wait for all pods to be ready
$ kubectl get pod -n u4a-system
NAME                                                          READY   STATUS    RESTARTS   AGE
bff-server-6c9b4b97f5-gqrx6                                   1/1     Running   0          45m
capsule-controller-manager-6cf656b98c-sjm5n                   1/1     Running   0          66m
cert-manager-756fd78bff-wb2vh                                 1/1     Running   0          76m
cert-manager-cainjector-64685f8d48-qg69v                      1/1     Running   0          76m
cert-manager-webhook-5c46d68c6b-f4dkh                         1/1     Running   0          76m
cluster-component-ingress-nginx-controller-5bd67897dd-5m9n7   1/1     Running   0          76m
kube-oidc-proxy-5f4598c77c-fzl5q                              1/1     Running   0          65m
oidc-server-85db495594-k6pkt                                  2/2     Running   0          65m
resource-view-controller-76d8c79cf-smkj5                      1/1     Running   0          66m
```
3. At the end of the helm install, it'll prompt you with some notes like below:
```
NOTES:
1. Get the  ServiceAccount token by running these commands:

  export TOKENNAME=$(kubectl get serviceaccount/host-cluster-reader -n u4a-system -o jsonpath='{.secrets[0].name}')
  kubectl get secret $TOKENNAME -n u4a-system -o jsonpath='{.data.token}' | base64 -d
```
Save the token and will use it to add the cluster later.

4. Open the host configured using ingress below:

`https://portal.<replaced-ingress-nginx-ip>.nip.io`

If your host isn't able to access nip.io, you should add the ip<->host mapping to your hosts file. Login with user admin/baas-admin (default one).

5. Prepare the environment
1) Create a namespace for cluster management, it should be 'cluster-system'.
```
kubectl create -n cluster-system
```

2) Add current cluster to the portal. Navigate to '集群管理' and '添加集群'
* for API Host, use the one from `hostK8sApiWithOidc`
* for API Token, use the one you saved from step 3.

Now, you should have a cluster and a 'system-tenant' and tenant management.

## 2. Install baas-component


## 3. Add more components
1. Install [kube-dashboard](./kube-dashboard/README.md) to integrate with u4a.

Refer to [kubernetes dashboard ](https://github.com/kubernetes/dashboard) for details.

2. Install [kubelogin](./kubelogin/README.md) to integrate with u4a

Refer to [kubelogin](https://github.com/int128/kubelogin) for details.
