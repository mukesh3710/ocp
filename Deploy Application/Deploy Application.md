## Deploy Application

### Project
Project is an organizational feature of OpenShift that allows you to group your resources. Some technologies call it by different names, such as; organization unit, resource group, and in Kubernetes, it is called namespace. And don’t forget that OpenShift is built on top of Kubernetes, so you will always get to see project, and namespace used interchangeably.
```yaml
oc projects # To see all the projects available on an OpenShift cluster
oc project # To see your current project 
oc new-project app-project # To create a new project
```
---
### Creating Pod In OpenShift Cluster
There are different ways a pod can be created in OpenShift. Just as we learnt in my Kubernetes course, a pod can be created as an individual object and by using the deployment. The same is applicable to OpenShift. Basically, to create a pod in OpenShift, we can:
- Create a pod as an individual object.
- Create a pod by using deployment.
- Create a pod by using deployment config (dc).
- Create a pod by using the new-app option

---
### Individual Object
To create a pod, with the name, “nginx-app”, as an individual object, use oc run command
```yaml
oc run nginx-app --image=nginx # To create a pod, with the name, “nginx-app”
oc get pods # To verify that the pod has been created
oc get pods -o wide # To get more information about the pod
oc describe pod nginx-app # To get a better information about the pod
oc delete pod nginx-app # To delete pod 

Login to core nodes form wide output: 
curl 10.217.0.70
sudo crictl ps # The default container runtime that the openshift uses is the CRIO
sudo crictl ps |grep nginx-app
```

---
### What Is Deployment In OpenShift/Kubernetes?
A deployment is a layer over a pod. It’s a collection of some sets of pods and other resources. Using deployment to create a pod manages the applications or pods in a holistic manner as the pods will be managed by deployment, accompanied with other resources and features that come with deployment, one of which is the replicaset.
```yaml
oc create deployment nginx-app --image=nginx
oc get deployment # To verify that the deployment has been created
oc get all #  To verify all resouce created by deployment 
oc get pods # To verify that the pod has been created

# If you want to get more information about the pod, you can use the commands we used above while we created a pod as a single/individual object.

oc adm policy add-scc-to-user anyuid -z default -n app-project # This allows the container to run as the root user. Use this cautiously,
```

---
### What Is Replicaset In OpenShift/Kubernetes
- A replicaset is a feature of an OpenShift/Kubernetes cluster that is responsible for the replication of a pod. If you want to increase or decrease the quantity of a pod, the replicaset allows you to do that, and with deployment, you don’t have to create the replicaset feature separately.
- The replicaset will automatically be created and managed by the deployment. With deployment, you will also have the benefit of doing an automatic update, pod update, and you can also easily roll back to the previous update if the update is not compatible or runs into issues.
- Creating a pod as a single object will not give you all these benefits of using deployment as mentioned above. To create a pod, as I’ve said, you can choose between creating a pod as an individual object and creating a pod using deployment. Most will agree that creating a pod with deployment is better and more realistic in a production environment than creating a pod as a single object.
- However, there are some tests you might want to quickly carry out, and in such cases, we can create a pod as an individual object. Having understood what deployment is, let’s look at what deployment config is.
```yaml
oc scale deployment nginx-app --replicas 4
```

---
### What Is Deployment Config (dc) & Replication Controller In OpenShift
- Deployment config is strictly an OpenShift object. It is similar to the deployment object. Both do the same thing using a slightly different method. For example, one of the features that comes with creating deployment is replicaset, while that of deployment config is replication controller.
- The replicaset and the replication controller do the same thing, which is managing the quantities of pods. The replicaset is simply the successor of the replication controller.
- A common question I’m often asked is, “Do I use deployment or do I use deployment config?” Well, Red Hat has advised that we should try as much as possible to always use deployment, because deployment serves as a descendant of the deployment config. However, if you need a specific feature of the deployment config object, you can make use of the deployment config. Otherwise, make use of deployment.
```yaml
oc create dc nginx-app --image=nginx
oc get dc
oc get pods
oc describe pod nginx-app-1-tnqrq
```

---
### Using The “oc new-app” Option
- A pod can be created by using the oc command with the “new-app” option. This is one of the easiest, and arguably the best way of creating a pod in an OpenShift cluster because the “oc new-app” command automatically creates a deployment which will be used to manage the pods.
- In the older version of OpenShift, the new-app option usually creates a deployment config, but in the newer version, it now creates a deployment. And if you are wondering why you should use the new-app option to create a pod when you can create a pod using the deployment straight away since the new-app option also creates deployment .
- The answer is that the new-app option does not only create deployment, it also creates service for you.
```yaml
oc new-app --image=nginx --name=nginx-app
oc get pods
oc get deployment
oc get service
oc describe service nginx-app
```

---
### Troubleshooting Containers and Pods
```yaml
oc describe pod pod-name # Gather Pod Information
oc get events -n namespace # View Events
oc rsh pod-name # Access Pod Shell
oc logs pod # Retrieve logs with oc logs.
oc exec # to run commands.

crictl ps --name <container_name>
crictl inspect <container_id>
```
---
### Tools for Image Management
```yaml

Skopeo - Manage images without a running container runtime:
skopeo inspect docker://<image>
skopeo copy docker://<source_image> docker://<destination_image>

oc image - OpenShift CLI tool for image management:
oc image info <image>
oc image extract <image> --path /path/to/extract
```
---
### Other Essential Commands
```yaml
oc delete all --all # To delete all the resources at once in a project
oc delete project app-project # To delete a project
oc adm must-gather #
oc adm inspect ns/app-project
oc new-app --image=nginx --name=nginx-app
oc adm policy add-scc-to-user anyuid -z default -n app-project
```

