
# AWX Deployment

## Create a minikube cluster for testing

If you do not have an existing cluster, the awx-operator can be deployed on a Minikube cluster for testing purposes.

```shell
minikube start --cpus=4 --memory=6g --addons=ingress
```

Once Minikube is deployed, check if the node(s) and kube-apiserver communication is working as expected.

```shell
minikube kubectl -- get nodes
```

## Deploy AWX Operator on Kubernetes

Clone operator deployment code

```shell
git clone https://github.com/ansible/awx-operator.git
cd awx-operator/
RELEASE_TAG=`curl -s https://api.github.com/repos/ansible/awx-operator/releases/latest | grep tag_name | cut -d '"' -f 4`
echo $RELEASE_TAG
git checkout $RELEASE_TAG
```

Create namespace where operator will be deployed

```shell
export NAMESPACE=awx
kubectl create ns ${NAMESPACE}
kubectl config set-context --current --namespace=$NAMESPACE 
```

And finally deploy awx-operator into your cluster
```shell
make deploy
```

Wait a few minutes and awx-operator should be running:
```shell
kg all -n awx
```

Uninstalling If you want to remove the awx-operator

```shell
export NAMESPACE=awx
make undeploy
```

## Create AWX Deployment

Create a file awx-demo.yaml

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
```

### Deploy AWX

```shell
kubectl apply -f awx-demo.yml
awx.awx.ansible.com/awx-demo created
```

### Get Admin Password

By default, the admin user is admin and the password is available in the <resourcename>-admin-password secret.

```shell
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
```


### Get the Path to Access Ansible AWX

You will find the NodePort and IP for service awx-demo-service

```shell
kubectl get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
awx-demo-postgres      ClusterIP   None             <none>        5432/TCP            23m
awx-demo-service       NodePort    10.110.146.161   <none>        80:31726/TCP        23m
awx-operator-metrics   ClusterIP   10.104.113.88    <none>        8383/TCP,8686/TCP   23m
kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP  
```

Access Ansible AWX Portal at IP_ADDRESS:NODE_PORT.

On minikube - get the minikube node ip and access AWX on port 31726

```shell
kubectl get node -o wide
```

http://192.168.59.105:30569

OR

Forward localhost (minkube VM Localhost) port 7080 -> 80

```shell
kubectl port-forward service/awx-demo-service 7080:80
```

http://localhost:30569
