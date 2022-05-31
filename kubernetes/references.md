## Some basic commands

```shell
# get a given resource form all namespaces
kubectl get {pods | deployments | services | ingress | ...}  --all-namespaces
```


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

## Monitoring

### Readiness & Livenes probes

Once created you must make sure a given POD is ready to receive requests (in case it's exposed via a service). Whithout a readiness probe kubernetes assumes that every POD is ready, this is going to avoid users to reach out a unresponsiveness POD.

The liveness probe ensures the container is staill alive and healthy in runtime.

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
      initialDelaySeconds: 20 
      periodSeconds: 5
      failureThreshold: 8

    livenessProbe:
      httpGet:
        path: /api/health
        port: 8080
      initialDelaySeconds: 20 
      periodSeconds: 5
      failureThreshold: 8
      
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

    livenessProbe:
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

## Multi container PODs

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

## Labels and Selectors

Use to categorize/classify different groups and objects in Kubernetes. They are very useful whenever we wanty to make a decision over a group of objects.
A ggod example is the usage of labels alongside replica sets:

```yaml
apiVersion: v1
kind: ReplicaSet
metadata: 
  name: myapp
  #Replica set labels
  labels:
    group: backend 
spec:
  replicas: 2
  selector:
    matchLabels:
      group: backend # this label value has to match with the one on template.metadata.lables.group
  template:
    metadata:
      #Those are the labels configured on the POD
      labels:
        group: backend
  containers:
  - name: myapp
    image: myappcontainer
    ports:
      - containerPort: 8080    
 
```

The following comamnd can be used to get all POD names using a given label

```shell
kubectl get pods --selector=labelName=labelValue -o jsonpath='{.items[*].metadata.name}
```

OR

```shell
kubectl get pods -l labelName=labelValue --no-headers
```



The following comamnd can be used to filter several labels (AND operator logic)

```shell
kubectl get pod --selector=group=backend --selector=owner=human-resources --selector=env=staging
```

## Deployments

### Creating from command line 

```shell
kubectl create deployment nginx --image=nginx:1.16
```

### Creating from file 

```yaml
apiVersion: apps/v1 # check always the api version for deployment, which is apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx # must match on with is within spec.selector.matchLabels.app value
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
More complete deployment example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: backend
spec:
  minReadySeconds: 20
  progressDeadlineSeconds: 600
  replicas: 4
  selector:
    matchLabels:
      name: webapp-backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: webapp-backend
    spec:
      containers:
      - image: my-backend-image:latest
        imagePullPolicy: IfNotPresent
        name: go-backend
        ports:
        - containerPort: 8080
          protocol: TCP
        terminationMessagePath: /var/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
```


### Rollout 

This is a very powerful way to ensure zero downtime updates (by default tha's the way a Deployment object is created):
The follwoing are useful commands to cehck rollout progress when a deplopyment is changed:

```shell
#check rollout status
kubectl rollout status deployment/deploymentName

#check rollout history
kubectl rollout history deployment/deploymentName --revision={1, 2, 3...} # -- revision is optinal and shows the status of ach revision

#rollback a rollout 
kubectl rollout undo deployment/deploymentName

```
## Jobs and Cron jobs

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob  
spec:
  template:
    spec:
      containers:
      - name: my-job
        image: my-job-image
      restartPolicy: Never
```
A cron job every day 10:30 

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  schedule: "30 10 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
           - name: my-job
             image: my-job-image
          restartPolicy: Never
```

## Services

Kubernetes services allow communication among different components inside and outside of a K8S applicaiton.
Services help to connect deployed kubernetes with other external apps and users.

Those are the service types provided by kubernetes:

### Node Port

Maps a port from the node to a port on the POD. By default the allowed ports are in the range of 30000 to 32767. Below an yaml example of a NodePort type service:

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: myapp
spec:
  type: NodePort
  ports:
    targetPort: 80
    port: 80
    nodePort: 30002
  selector:
    ## use the labels used on the POD to make the connection to the respective service
 
```

> IMPORTANT: NodePort type exposes a direct connection to a POD. Be VERY careful with such configuration as it is not the most secure one.

Cluster IP
Load Balancer


## Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. 
Traffic routing is controlled by rules defined on the Ingress resource (source [here](https://kubernetes.io/docs/concepts/services-networking/ingress/)).

```yaml
apiVersion: v1
items:
- apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    generation: 1
    name: ingress-ecommerce-app
    namespace: ecommerce
  spec:
    rules:
    - http:
        paths:
        - backend:
            service:
              name: book-store-service
              port:
                number: 8080
          path: /books
          pathType: Prefix
        - backend:
            service:
              name: wear-store-service
              port:
                number: 8080
          path: /wear
          pathType: Prefix
        - backend:
            service:
              name: payment-service
              port:
                number: 8080
          path: /payment
          pathType: Prefix         
```
