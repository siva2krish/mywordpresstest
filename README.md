
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
# experiment

**Based on my recent experience using helm in advance level by converting Redhat OpenShift Templates to Helm charts,
  it is quite good experience to end with a sucessfull usecase**

**Openshift code for nginx service deployment**

```
apiVersion: v1	
		kind: Template
		metadata:
		  creationTimestamp: null
		  name: lagoon-openshift-template-nginx
		parameters:
		  - name: SERVICE_NAME
		    description: Name of this service
		    required: true
		  - name: SAFE_BRANCH
		    description: Which branch this belongs to, special chars replaced with dashes
		    required: true
		  - name: SAFE_PROJECT
		    description: Which project this belongs to, special chars replaced with dashes
		    required: true
		  - name: BRANCH
		    description: Which branch this belongs to, original value
		    required: true
		  - name: PROJECT
		    description: Which project this belongs to, original value
		    required: true
		  - name: LAGOON_GIT_SHA
		    description: git hash sha of the current deployment
		    required: true
		  - name: SERVICE_ROUTER_URL
		    description: URL of the Router for this service
		    value: ''
		  - name: OPENSHIFT_PROJECT
		    description: Name of the Project that this service is in
		    required: true
		  - name: REGISTRY
		    description: Registry where Images are pushed to
		    required: true
		  - name: DEPLOYMENT_STRATEGY
		    description: Strategy of Deploymentconfig
		    value: 'Rolling'
		  - name: SERVICE_IMAGE
		    description: Pullable image of service
		    required: true
		  - name: CRONJOBS
		    description: Oneliner of Cronjobs
		    value: ''
		objects:
		  - apiVersion: v1
		    kind: DeploymentConfig
		    metadata:
		      creationTimestamp: null
		      labels:
		        service: ${SERVICE_NAME}
		        branch: ${SAFE_BRANCH}
		        project: ${SAFE_PROJECT}
		      name: ${SERVICE_NAME}
		    spec:
		      replicas: 1
		      selector:
		        service: ${SERVICE_NAME}
		      strategy:
		        type: ${DEPLOYMENT_STRATEGY}
		      template:
		        metadata:
		          creationTimestamp: null
		          labels:
		            service: ${SERVICE_NAME}
		            branch: ${SAFE_BRANCH}
		            project: ${SAFE_PROJECT}
		        spec:
		          tolerations:
		            - effect: NoSchedule
		              key: autoscaled
		              operator: Equal
		              value: 'true'
		          containers:
		            - image: ${SERVICE_IMAGE}
		              name: ${SERVICE_NAME}
		              ports:
		                - containerPort: 8080
		                  protocol: TCP
		              readinessProbe:
		                httpGet:
		                  path: /nginx_status
		                  port: 50000
		                initialDelaySeconds: 5
		                timeoutSeconds: 3
		              livenessProbe:
		                httpGet:
		                  path: /nginx_status
		                  port: 50000
		                initialDelaySeconds: 90
		                timeoutSeconds: 3
		                failureThreshold: 5
		              envFrom:
		                - configMapRef:
		                    name: lagoon-env
		              env:
		                ## LAGOON_GIT_SHA is injected directly and not loaded via `lagoon-env` config
		                ## This will cause the cli to redeploy on every deployment, even the files have not changed
		                - name: LAGOON_GIT_SHA
		                  value: ${LAGOON_GIT_SHA}
		                - name: SERVICE_NAME
		                  value: ${SERVICE_NAME}
		                - name: CRONJOBS
		                  value: ${CRONJOBS}
		              resources:
		                requests:
		                  cpu: 10m
		                  memory: 10Mi
		      test: false
		      triggers:
		        - type: ConfigChange
		    status: {}
```
**Helm code after converted from OpenShift template for nginx service**

```

apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: {{ template "fullname" . }}
	  labels:
	    app: {{ template "name" . }}
	    chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
	    release: {{ .Release.Name }}
	    heritage: {{ .Release.Service }}
	spec:
	  replicas: {{ .Values.replicaCount }}
	  template:
	    metadata:
	      annotations:
	        appuio.ch/backupcommand: /bin/sh -c "/bin/busybox tar -cf - -C {{.Values.PERSISTENT_STORAGE_PATH}} ."
	        backup.appuio.ch/file-extension: .{{.Values.SERVICE_NAME}}.tar
	      labels:
	        app: {{ template "name" . }}
	        release: {{ .Release.Name }}
	    spec:
	      volumes:
	        - name: {{.Values.SERVICE_NAME}}
	          persistentVolumeClaim:
	            claimName: {{ .Release.Name }}-store
	      containers:
	        - name: {{.Values.SERVICE_NAME}} 
	          image: {{ .Values.image.nginxRepository }}
	          imagePullPolicy: {{ .Values.image.pullPolicy }}
	          ports:
	            - containerPort: 8080
	              protocol : TCP
	          readinessProbe:
	            httpGet:
	              path: /nginx_status
	              port: 50000
	            initialDelaySeconds: 120
	            timeoutSeconds: 3
	          livenessProbe:
	            httpGet:
	              path: /nginx_status
	              port: 50000
	            initialDelaySeconds: 120
	            timeoutSeconds: 3
	            failureThreshold: 5
	          envFrom:
	            - configMapRef:
	                name: lagoon-env 
	          env:
	            - name: LAGOON_GIT_SHA
	              value: {{.Values.LAGOON_GIT_SHA}}
	            - name: NGINX_FASTCGI_PASS       
	              value: '127.0.0.1'
	          env:
	            - name: SERVICE_NAME
	              value: {{.Values.SERVICE_NAME}}-svc
	            - name: NGINX_FASTCGI_PASS       
	              value: '127.0.0.1'
	            - name: SAFE_BRANCH
	              value: "{{.Values.SAFE_BRANCH}}"
	            - name: SAFE_PROJECT
	              value: "{{.Values.SAFE_PROJECT}}"
	            - name: BRANCH
	              value: "{{.Values.BRANCH}}"
	            - name: PROJECT
	              value: "{{.Values.PROJECT}}"
	            - name: LAGOON_GIT_SHA
	              value: "{{.Values.LAGOON_GIT_SHA}}"
	            - name: SERVICE_ROUTER_URL
	              value: ''
	            - name: OPENSHIFT_PROJECT
	              value: "{{.Values.OPENSHIFT_PROJECT}}"
	            - name: REGISTRY
	              value: "{{.Values.REGISTRY}}"
	            - name: DEPLOYMENT_STRATEGY
	              value: "{{.Values.DEPLOYMENT_STRATEGY}}"
	            - name: SERVICE_IMAGE
	              value: "{{.Values.SERVICE_IMAGE}}"
	            - name: NGINX_SERVICE_IMAGE
	              value: "{{.Values.NGINX_SERVICE_IMAGE}}"
	            - name: PHP_SERVICE_IMAGE
	              value: "{{.Values.PHP_SERVICE_IMAGE}}"
	            - name: NGINX_SERVICE_NAME
	              value: "{{.Values.NGINX_SERVICE_NAME}}"
	            - name: PHP_SERVICE_NAME
	              value: "{{.Values.PHP_SERVICE_NAME}}"
	            - name: CRONJOBS
	              value: "{{.Values.CRONJOBS}}"
	            - name: PERSISTENT_STORAGE_PATH
	              value: "{{.Values.PERSISTENT_STORAGE_PATH}}"
	            - name: PERSISTENT_STORAGE_CLASS
	              value: "{{.Values.PERSISTENT_STORAGE_CLASS}}"
	            - name: PERSISTENT_STORAGE_SIZE
	              value: "{{.Values.PERSISTENT_STORAGE_SIZE}}"
	            # - name: ENVIRONMENT_TYPE
	            #   value: "{{.Values.ENVIRONMENT_TYPE}}"
	            # - name: OPENSHIFT_NAME
	            #   value: "{{.Values.OPENSHIFT_NAME}}"
	            # - name: MONITORING_URLS
	            #   value: "{{.Values.MONITORING_URLS}}"
	            # - name: SERVICE_IMAGE
	            #   value: "{{.Values.SERVICE_IMAGE}}"
	          volumeMounts:
	            - name: {{.Values.SERVICE_NAME}}
	              mountPath: {{.Values.PERSISTENT_STORAGE_PATH}}
	          resources:
	{{ toYaml .Values.resources | indent 12 }}
	        - name: php
	          image: {{ .Values.image.phpRepository }}
	          imagePullPolicy: {{ .Values.image.pullPolicy }}
	          ports:
	            - containerPort: 9000
	              protocol : TCP
	          readinessProbe:
	            exec:
	              command:
	                - /usr/sbin/check_fcgi
	            initialDelaySeconds: 2
	            periodSeconds: 5
	          livenessProbe:
	            exec:
	              command:
	                - /usr/sbin/check_fcgi
	            initialDelaySeconds: 60
	            periodSeconds: 5
	          envFrom:
	            - configMapRef:
	                name: lagoon-env 
	          env:
	            - name: LAGOON_GIT_SHA
	              value: {{.Values.LAGOON_GIT_SHA}}
	            - name: SERVICE_NAME       
	              value: {{.Values.SERVICE_NAME}}-svc
	            - name: CRONJOBS
	              value: {{.Values.CRONJOBS}}
	          env:
	            - name: SERVICE_NAME
	              value: {{.Values.SERVICE_NAME}}-svc
	            - name: SAFE_BRANCH
	              value: "{{.Values.SAFE_BRANCH}}"
	            - name: SAFE_PROJECT
	              value: "{{.Values.SAFE_PROJECT}}"
	            - name: BRANCH
	              value: "{{.Values.BRANCH}}"
	            - name: PROJECT
	              value: "{{.Values.PROJECT}}"
	            - name: LAGOON_GIT_SHA
	              value: "{{.Values.LAGOON_GIT_SHA}}"
	            - name: SERVICE_ROUTER_URL
	              value: ''
	            - name: OPENSHIFT_PROJECT
	              value: "{{.Values.OPENSHIFT_PROJECT}}"
	            - name: REGISTRY
	              value: "{{.Values.REGISTRY}}"
	            - name: DEPLOYMENT_STRATEGY
	              value: "{{.Values.DEPLOYMENT_STRATEGY}}"
	            - name: SERVICE_IMAGE
	              value: "{{.Values.SERVICE_IMAGE}}"
	            - name: NGINX_SERVICE_IMAGE
	              value: "{{.Values.NGINX_SERVICE_IMAGE}}"
	            - name: PHP_SERVICE_IMAGE
	              value: "{{.Values.PHP_SERVICE_IMAGE}}"
	            - name: NGINX_SERVICE_NAME
	              value: "{{.Values.NGINX_SERVICE_NAME}}"
	            - name: PHP_SERVICE_NAME
	              value: "{{.Values.PHP_SERVICE_NAME}}"
	            - name: CRONJOBS
	              value: "{{.Values.CRONJOBS}}"
	            - name: PERSISTENT_STORAGE_PATH
	              value: "{{.Values.PERSISTENT_STORAGE_PATH}}"
	            - name: PERSISTENT_STORAGE_CLASS
	              value: "{{.Values.PERSISTENT_STORAGE_CLASS}}"
	            - name: PERSISTENT_STORAGE_SIZE
	              value: "{{.Values.PERSISTENT_STORAGE_SIZE}}"
	            # - name: ENVIRONMENT_TYPE
	            #   value: "{{.Values.ENVIRONMENT_TYPE}}"
	            # - name: OPENSHIFT_NAME
	            #   value: "{{.Values.OPENSHIFT_NAME}}"
	            # - name: MONITORING_URLS
	            #   value: "{{.Values.MONITORING_URLS}}"
	            # - name: SERVICE_IMAGE
	            #   value: "{{.Values.SERVICE_IMAGE}}"
	          volumeMounts:
	            - name: {{.Values.SERVICE_NAME}}
	              mountPath: {{.Values.PERSISTENT_STORAGE_PATH}}
	          resources:
	{{ toYaml .Values.resources | indent 12 }}       
	    {{- if .Values.nodeSelector }}
	      nodeSelector:
	{{ toYaml .Values.nodeSelector | indent 8 }}
	    {{- end }}

```





 
