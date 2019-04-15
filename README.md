
# Helm Blog
**Motivation**
```
OpenShift is an application platform built around a Kubernetes core. Distilling decades of experience with container clusters, 
Kubernetes has quickly become the defacto system for orchestrating containers on Linux. 
OpenShift employs standard Kubernetes mechanisms to maintain compatibility with both interfaces and techniques, 
but as an enterprise distribution, ready for production deployment and day-zero support of critical applications, 
it extends and diverges from “vanilla” Kubernetes in some ways.
Where OpenShift differs, it is often where no equivalent feature existed in the upstream core version at the time Red Hat needed 
to deliver it. OpenShift Routes, for example, predate the related Ingress resource that has since emerged in upstream Kubernetes. 
In fact, Routes and the OpenShift experience supporting them in production environments helped influence the later Ingress design,
and that’s exactly what participation in a community like Kubernetes is all about.
I was curious about some of Openshift’s extended features. In particular, 
I wanted to explore the differences between Kubernetes Deployments and OpenShift’s Deployment Configurations.
```
**What is Helm?**
```
If you’re already familiar with Helm you can skip this section and scroll down to Creating new charts and keep reading. Helm is the de facto application for management on Kubernetes. It is officially a CNCF incubator project.
“Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.” - https://helm.sh/
Helm’s operation is based on cooperation between two main components: a command line tool called helm, and a server component called tiller, which has to run on the cluster it manages.
The main building block of Helm based deployments are Helm Charts: these charts describe a configurable set of dynamically generated Kubernetes resources. The charts can either be stored locally or fetched from remote chart repositories.
```
**INSTALLING HELM**
```
To install the helm command use a supported package manager, or simply download the pre-compiled binary:
# Brew
$ brew install kubernetes-helm
# Choco
$ choco install kubernetes-helm
# Gofish
$ gofish install helm
Installing Tiller
Note: When RBAC is enabled on your cluster you may need to set proper permissions for the tiller pod.
To start deploying applications to a pure Kubernetes cluster you have to install tiller with the helm init command of the CLI tool.
# Select the Kubernetes context you want to use
$ kubectl config use-context docker-for-desktop
$ helm init
That’s it, now we have a working Helm and Tiller setup.
```
Github 
 

Based on my recent experience using helm in advance level by converting Redhat OpenShift Templates to Helm charts, it is quite good experience to end with successful use cases 


 
