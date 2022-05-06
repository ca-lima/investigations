# useful-commands

This is to be used for my internal consumption and for reference when I need to run some commands hard to memorize

## AWS

### Autoscaling groups

#### Complete lifecycle event 
```
aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --instance-id <id> --lifecycle-hook-name <InstanceTerminateHook|InstanceLaunchHook> --auto-scaling-group-name <lima-dev-AutoScalingGroup-1GJX181NW78AT>
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




