# Kubernetes Nitty Gritty

Practice is done with minikube
- Original tooling to run Kubernetes locally
- Abstracts complexities while getting started
- Powerful add-on features

`kubectl get all`
Returns all resources contained in the cluster

`kubectl get deployment`
limits the previous search to only deployments

`kubectl get deployment/helloworld -o yaml`
Outputs the information on the specified `helloworld` deployment as a YAML document

`kubectl get service/helloworld -o yaml`
Same as previous command except with a service

`kubectl get pods`
Shows all pods on a system

`kubectl delete all --all`
Please be careful
This deletes everything

`minikube start` | `minikube stop` | `minikube status`
Commands for managing our minikube instance, which we are using to practice Kubernetes

`minikube addons list`
Lists all of the addons associated with minikube

`minikube addons enable <addon>`

`kubectl create -f helloworld-all.yml`
Something like this can be used with the triple dash notation to shorten the process of deploying a deployment, service, and the replicaset in one go.

This is an example YAML file that creates a `helloworld` deployment along with its accompanying service and replica set all in one file.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-all-deployment
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: helloworld-all-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: helloworld
```

## Upgrades
`kubectl create -f helloworld-black.yaml --record`
This is the same as the basic create command but the --record flag makes it so that rollout history is collected

`kubectl set image deployment/navbar-deployment helloworld=karthequian/helloworld:blue`
Changes the image of the deployment to a new one that gets specified in the later half of the command

`kubectl rollout history deployment/navbar-deployment`
Seeing deployment rollout history

`kubectl rollout undo deployment/navbar-deployment`
rolls back deployment to the previous rollout

---
## Labels
Almost anything here can be transposed to services and deployments

`kubectl get pods --show-labels`
Same command as before but shows labels as well

`kubectl label po/helloworld app=helloworldapp`
This would add the label `app=helloworldapp` to the existing helloworld pod

`kubectl label po/helloworld app=helloworldapp --overwrite`
To edit an existing label, you can use this command. Specifically, the `overwrite` flag to overwrite existing labels

`kubectl label pod/helloworld app-`
You can use this to delete a label. State the resource, in this case it is `pod/helloworld` and then include the label key followed by a `-`

`kubectl get pods --selector env=production`
This command would look at all the pods and display pods that have the label/selector of "production"

`kubectl get pods --selector env=production --show-labels`
This command shows all the pods with the label env set to production and shows all the labels associated with the filtered field

`kubectl get pods --selector dev-lead=karthik,env=staging`
This is searching for a pod by selecting multiple labels as the filter field. In this case it is dev-lead karthik and environment staging

`kubectl get pods --selector dev-lead!=karthik,env!=staging`
This is the syntax for searching for not equal label searching

`kubectl get pods -l 'release-version in (1.0,2.0)'`
Something like this would get the release versions between 1.0 and 2.0. The main thing to look at is the in operator

`kubectl delete pods -l dev-lead=Brian`
This command deletes all the pods that have the label `dev-lead=Brian`
Mass delete by labels

#### Configmaps
`kubectl create configmap logger --from-literal=log_level=debug`
Here we created a configmap with the name of `logger` and there is a literal key value set to `log_level=debug`.

This is an example of the configuration in the spec section of a deployment yaml file
>-name: log_level
	valueFrom:
		configMapKeyRef
			name: logger # Read from a configmap called log_level
			key: log_level # Read the get called Log_level


#### Health Checks

`readinessProbe` : Length of time to wait for pod to initialize after pod startup, before applying health check
`livenessProbe` : Length of time to wait for a pod to initialize after pod startup, before applying health check

## Troubleshooting
`kubectl describe deployment <deployment-name>`
Describes the whole deployment

`kubectl describe po/<pod-name>`
Gets information on the whole pod
Contains an events tab that you can look to get why it broke

`kubectl logs <deployment-name>`
Shows all the logs for that specific pod

`kubectl exec -it <podname> /bin/bash`
You can use this to look into the pod

`kubectl exec -it <podname> -c <container-name> /bin/bash`
This is the same as the previous command but lets you go into a specific container in the pod. This is used when you have pods with many containers


## Kubernetes Dashboard
![[Pasted image 20220725131324.png]]

![[Pasted image 20220725131346.png]]
![[Pasted image 20220725131358.png]]
![[Pasted image 20220725131443.png]]
![[Pasted image 20220725131454.png]]
![[Pasted image 20220725131520.png]]
Also holds event data which is very useful

You can also look at pods specifically for more in depth data
![[Pasted image 20220725131629.png]]
![[Pasted image 20220725131640.png]]
![[Pasted image 20220725131708.png]]
You can also start a shell for the pod through the dashboard

You can also create resources from the dashboard. You can create from inputted YAML, you can create by passing in a YAML file or form a form.
![[Pasted image 20220725131906.png]]
![[Pasted image 20220725131939.png]]
![[Pasted image 20220725132015.png]]
