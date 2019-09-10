# Lab2 - Secure Guestbook API with API Connect and AppID

* Create API Connect Draft
* Configure DataPower Gateway
* Publish Guestbook API 
* Secure Guestbook API with Ingress Server
* Use AppID to authenticate users

1. Create an instance of API Connect on IBM Cloud

	* Sign in to https://cloud.ibm.com/,
	* Go to the [Catalog](https://cloud.ibm.com/catalog),
	* Search the catalog and filter by `api connect`, or filter by Category `Integration`,
	* Click the `API Connect` service,
	* Click `Create`,

2. Create a new API Draft

	* Go to the API Connect Dashboard,
	* Select the default catalog `Sandbox`,
	* From the Home menu, go to `Drafts` > `APIs`
	* Click the `Add` button > select `Import API from a file or URL` > select `Or import from URL`,
	* In the `URL` enter the URL to your published OpenAPI spec file, e.g. http://169.63.218.104:32145/openapi.yaml
	* Click `Import`,
	* This will create an `API Draft` in the API Designer,
	* Click the `validate` icon in the top right,

3. Edit the Draft API

	* Change the Title to `Guestbook API`,
	* Change the Name to `guestbook-api`,
	* Go to Scheme and check `http`,
	* 

4. Create a new Product

	* In the API Designer, in the Draft API, in the top right, click the menu drop down > click the `Generate a default product` option,
	* Change the title to `Guestbook API`,
	* click the `Create product`,
	* Click `All APIs` and go to `Products`,
	* Click the `Guestbook API 1.0.0` to edit the product,
	* 


4. Add Assembly to API

	* In the API Designer, in the API Draft, go to the `Assemble` tab,
	* Click the `Create assembly` button,
	* Make sure `DataPower Gateway policies` is selected,
	* Under `Policies`, drag the `Invoke` action onto the assembly,
	* Configure the `Invoke` action, and add the URL for the Guestbook API service on Kubernetes, e.g. http://169.63.218.104:32145,
	* Save the Draft API,
	* Test the Draft API, click the `play` icon to test,
	* A `Setup` window will open on the left side,
	* Because we do not have a product yet, enter a name `Guestbook API` in the `Or create a new product and publish it to the selected catalog` section,
	* Click the `Create and publish` button

```
# Source: web-terminal/templates/web-terminal-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: terminal-ingress
  annotations:
    ingress.bluemix.net/rewrite-path: "serviceName=terminal-service-1 rewrite=/;serviceName=terminal-service-2 rewrite=/;serviceName=terminal-service-3 rewrite=/;"
    ingress.bluemix.net/proxy-connect-timeout: "serviceName=terminal-service-1 timeout=75s;serviceName=terminal-service-2 timeout=75s;serviceName=terminal-service-3 timeout=75s;"
    ingress.bluemix.net/proxy-read-timeout: "serviceName=terminal-service-1 timeout=3600s;serviceName=terminal-service-2 timeout=3600s;serviceName=terminal-service-3 timeout=3600s;"
    kubernetes.io/ingress.class: nginx
spec:
  tls:
  - hosts:
    - testing-cluster.us-east.containers.appdomain.cloud
    secretName: testing-cluster
  rules:
  - host: testing-cluster.us-east.containers.appdomain.cloud
    http:
      paths:
      - path: "/term1"
        backend:
          serviceName: "terminal-service-1"
          servicePort: 80
      - path: "/term2"
        backend:
          serviceName: "terminal-service-2"
          servicePort: 80
      - path: "/term3"
        backend:
          serviceName: "terminal-service-3"
          servicePort: 80
```