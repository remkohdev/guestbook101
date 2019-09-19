# Lab1 - Deploy Guestbook API App to Kubernetes using Bash

1.  Build using Bash scripts,

	* IN the project root directory, create a bash script `docker-build-tag-push.sh` and paste the code below. Replace `<username>` by your Docker Hub username. This code will build, tag and push an image to the Docker Hub public repository,

		```bash
		echo '=====>build guestbook-api<====='
		docker build --no-cache -t guestbook-api .

		echo '=====>tag guestbook-api<====='
		docker tag guestbook-api:latest <username>/guestbook-api:0.1.0

		echo '=====>push guestbook-api<====='
		docker push <username>/guestbook-api:0.1.0
		```

	* Login to Docker Hub with your Docker Hub username and run the script `docker-build-tag-push.sh`,
	
		```console
		$ docker login -u <username>
		$ sh docker-build-tag-push.sh
		```

2.  Add Kubernetes object specs, 

	* Create a new directory `helm/templates`,

		```console
		$ mkdir -p helm/templates
		```

	* Create a `helm/templates/deployment.yaml`,

		```yaml
		apiVersion: apps/v1beta2
		kind: Deployment
		metadata: 
		  name: guestbook-api-deployment
		  namespace: guestbook-ns
		  labels: 
			app: guestbook-api
		spec:
		  replicas: 1
		  selector: 
			matchLabels:
		  	  app: guestbook-api
		  template: 
			metadata: 
			  labels:
				app: guestbook-api
			spec: 
			  containers:
			  - name: guestbook-api
				image: <username>/guestbook-api:0.1.0
				ports:
				- name: main
				  protocol: TCP
				  containerPort: 3000
				envFrom:
				- configMapRef:
					name: guestbook-api-configmap
				resources:
				  requests:
				    memory: "120M"
					cpu: "500m" 
		```

	* Make sure your change the line `image: <username>/guestbook-api:0.1.0` in the `deployment.yaml` above by the Docker Hub username,

	* Create a `helm/templates/svc.yaml`,

		```yaml
		apiVersion: v1
		kind: Service
		metadata:
		  name: guestbook-api-svc
		  namespace: guestbook-ns
		  labels:
			app: guestbook-api
		spec:
		  type: NodePort
		  ports:
		  - name: main
			protocol: TCP
			port: 3000
			targetPort: 3000
		  selector: 
			app: guestbook-api
		```

	* Create a `helm/templates/configmap.yaml`,

		```yaml
		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: guestbook-api-configmap
		  namespace: guestbook-ns
		  labels:
    		app: guestbook-api
		data:
		  NODE_ENV: integration
		```

	* Create a `helm/templates/hpa.yaml`,

		```yaml
		apiVersion: autoscaling/v1
		kind: HorizontalPodAutoscaler
		metadata:
		  name: guestbook-api-hpa
		  namespace: guestbook-ns
		  labels: 
    		app: guestbook-api
		spec:
		  maxReplicas: 10
		  minReplicas: 2
		  scaleTargetRef:
			apiVersion: extensions/v1beta1
			kind: Deployment
			name: guestbook-api-deployment
		  targetCPUUtilizationPercentage: 80
		```

3.  Deploy using a Bash script,

	* Login to your Kubernetes cluster, e.g. for IBM Cloud see your cluster dashboard's Access tab, which should show you something close to the following instructions,

		```console
		$ ibmcloud login -a cloud.ibm.com -r us-south -g Default
		Email> user1@email.com

		Password> 
		Authenticating...
		OK

		Select an account:
		1. user1@email.com
		... and more

		$ ibmcloud target --cf
		```

	* Configure the current-context of kubectl to match your Kubernetes cluster, e.g. for IBM Cloud,

		```console
		$ ibmcloud ks cluster-config --cluster <cluster-id>

		$ export KUBECONFIG=/Users/user1/.bluemix/plugins/container-service/clusters/<cluster-id>/kube-config-dal10-<username>-<cluster>.yml
		```

	* In the project root directory, create a `kube-deploy.sh` bash script and copy paste the following code,

		```bash
		echo '=====>delete namespace guestbook-ns'
		kubectl delete namespace guestbook-ns
		echo '=====>create namespace guestbook-ns'
		kubectl create namespace guestbook-ns

		echo '=====>delete guestbook-api-configmap'
		kubectl delete configmap --namespace guestbook-ns guestbook-api-configmap
		echo '=====>create guestbook-api-configmap'
		kubectl create --namespace guestbook-ns -f ./helm/templates/configmap.yaml

		echo '=====>delete guestbook-api-deployment<====='
		kubectl delete deployment --namespace=guestbook-ns guestbook-api-deployment
		rc=$(eval 'kubectl get deployment -n guestbook-ns guestbook-api-deployment')
		while [ ! -z "$rc" ] 
		do
			rc=$(eval 'kubectl get deployment -n guestbook-ns guestbook-api-deployment')
		done
		echo '=====>create guestbook-api-deployment<====='
		kubectl create -f ./helm/templates/deployment.yaml

		echo '=====>delete guestbook-api-svc<====='
		kubectl delete svc --namespace=guestbook-ns guestbook-api-svc
		rc=$(eval 'kubectl get svc -n guestbook-ns guestbook-api-svc')
		while [ ! -z "$rc" ] 
		do
			rc=$(eval 'kubectl get svc -n guestbook-ns guestbook-api-svc')
		done
		echo '=====>create guestbook-api-svc<====='
		kubectl create -f ./helm/templates/svc.yaml

		echo '=====>delete guestbook-api-hpa<====='
		kubectl delete hpa --namespace=guestbook-ns guestbook-api-hpa
		rc=$(eval 'kubectl get svc -n guestbook-ns guestbook-api-hpa')
		while [ ! -z "$rc" ] 
		do
			rc=$(eval 'kubectl get svc -n guestbook-ns guestbook-api-hpa')
		done
		echo '=====>create guestbook-api-hpa<====='
		kubectl create -f ./helm/templates/hpa.yaml
		```

	* Run the deployment script,

		```bash
		$ sh kube-deploy.sh
		```

	* Your Guestbook API service should now be available via the Public IP of the Cluster and the NodePort of your Guestbook API service, e.g. http://<Public IP>:32300, 

		```console
		$ curl -X GET http://169.63.218.104:32300/
		{"started":"2019-09-13T03:39:17.923Z","uptime":95.718}
		```
	* Or via the Ingress subdomain and service port, e.g. http://<cluster-name>.<region>.containers.appdomain.cloud:<service-port>,

		```console
		$ curl -X GET http://remkohdev-cluster-3w.us-south.containers.appdomain.cloud:31356/
		{"started":"2019-09-19T02:45:32.243Z","uptime":556.014}
		```