# useful-commands

This is to be used for my internal consumption and for reference when I need to run some commands hard to memorize

## AWS

### Autoscaling groups

#### Complete lifecycle event 
```
aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --instance-id <id> --lifecycle-hook-name <InstanceTerminateHook|InstanceLaunchHook> --auto-scaling-group-name <lima-dev-AutoScalingGroup-1GJX181NW78AT>
```

Using Ec2 instance connect to make ssh tunneling
```
pip3 install ec2instanceconnectcli
mssh -f -N -L 5435:<rds endpoint>:5432 <bastion instanceId> -v
```

### STS

#### Decode mesasge

```
# Use when error messages are received and need to be decoded.
aws sts decode-authorization-message --encoded-message <message>
```

### Linux Commands

#### Journalctl - check system logs

```
#cloud-init related logs
journalctl -u cloud-init
```
```
#cloud-final related logs
journalctl -u cloud-final
```
#### Service status
```
#detailed service status
service <service-name> status
```

## Kubernetes

### Imperative commands

Useful tool that helps in getting one time tasks done quickly, as well as generate a definition template easily.

#### Examples:

Create a POD manifiest definition yaml output
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Generate Deployment with 2 replicas YAML 

```
kubectl create deployment --image=nginx nginx --dry-run=client --replicas=-2 -o yaml
```

Create a  ClusterIP service and expose pod redis on port 6379

```
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

### Helm

Helm diff (needs to install hel diff plugin)
```
helm diff upgrade <service> . -f <values filename>.yaml
```

Helm diff (traaefik example)
```
helm diff upgrade --allow-unreleased -n kube-system traefik traefik/traefik -f values.yaml -f <other values filename>.yaml
```

Helm install
```
helm upgrade --install <service> . -f values.yaml
```


### Postgres

Add owner to a table
```
ALTER TABLE policy OWNER TO policyservice;
```
