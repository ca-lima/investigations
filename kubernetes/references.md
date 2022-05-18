## Working with Pods

### Creating Pods

Reference link to [kubectl run documentation](https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl_run/)

Dry run

```shell
kubectl run <pod-name> --dry-run=client --image=nginx --restart=Never
```

Using yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 8080
          protocol: TCP

```

### Extracting Pods information

```shell
#Describes a given pod
kubectl describe pod <pod-name>

#Gets tabular pod information 
kubectl get pod <pod-name> -o wide

#Displays pod's output in yaml format (good to get output data to a file)
kubectl get pod <pod-name> -o yaml

```


