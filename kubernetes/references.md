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

### See logs from PODs

```shell
kubectl -n <namespace> logs <pod-name>
```

### Execute a command inside a POD

```shell
# COMMAND might be anything like: cat /var/log/logs or ls /usr/local
kubectl -n <namespace> exec <pod-name> -- [COMMAND]
```

### Mounting volumes

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myapp
  labels:
    name: myapp
spec:
  containers:    
  - name: logger
    image: mylogImag    
    volumeMounts:
    - mountPath: /var/log/messages/
      name: msg-volume
```

### Readiness probe

Once created uyou must make sure a given POD is ready to receive requests (in case it's exposed via a service). Whithout a readiness probe kubernetes assumes that every POD is ready, this is going to avoid users to reach out a unresponsiveness POD 

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myapp
  labels:
    name: myapp
spec:
  containers:    
  - name: myapp
    image: myAppImg    
    ports:
      - containerPort: 8080
    readinesProbe:
      httpGet:
        path: /api/health
        port: 8080      
```

Other ways to do a readiness probe:

```yaml

#via tcp socket

spec:
  containers:    
  - name: myapp
    image: myAppImg    
    ports:
      - containerPort: 8080
    readinesProbe:
      tcpSocket:
        port: 3306
```

```yaml

#via commands

spec:
  containers:    
  - name: myapp
    image: myAppImg    
    ports:
      - containerPort: 8080
    readinesProbe:
      command:
        - cat
        - /myapp/isHealthy
```



### Multi container PODs

PODs that share the same lifecycle:
 - Cresated together
 - Destroyed together
 - Share the same network
 - Have cess to the same volume storage

The folloing paterns are used for mult-container PODs:

### Sidecar

Imagine that we need a container respoinsible to get some telemetry/log/whjatever data from inside the POD and send it to another service. A Sidecar container could be used for such purpose:

#### Adapter

Imagine that before sending that information out you need to make a processing logic to transform the data somehow. In this case the adapter pattern is the best choice.

#### Ambassador

Servers as a proxy to make a transparent transformation to some connection/resource (for instance) in the sense there is no need to change POD's  data based on a given environment. Imagine you have a POD that is deployed on different environments and you need to communicate to a different API for each environment, the ambassador patern is a pattern to use on such scenario.


```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: myappcontainer
    ports:
      - containerPort: 8080
      
  - name: logger
    image: mylogImag
 
```



