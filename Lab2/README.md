# Lab2 - Secure the Guestbook API with API Connect

1. Create an instance of API Connect on IBM Cloud

	* Sign in to https://cloud.ibm.com/,
	* Go to the [Catalog](https://cloud.ibm.com/catalog),
	* Search the catalog and filter by `api connect`, or filter by Category `Integration`,
	* Click the `API Connect` service,
	* Click `Create`,

2. Create a new API Draft

	* Go to the API Connect Dashboard,
	* Select the default catalog `Sandbox`,
	* From the Home menu, select `Drafts`,

		![Home Dashboard](../images/apic-home-dashboard.png)

	* Select the `APIs` tab,
	* Click the `Add` button > select `Import API from a file or URL` > click the `Select File` button,
	* Browse to your project and select the `api/guestbook-api-swagger.json` file, and select `Open`,
	* Click `Import`,
	* This will create an `API Draft` in the API Designer,
	* Click the `validate` icon in the top right,

		![Imported API](../images/apic-imported-api.png)

3. Edit the Draft API

	* Review the Title to be `Guestbook API`,
	* Review the Name to be `guestbook-api`,
	* Review the Version to be `1.0.0`,
	* Go to Scheme and make sure the `https` is checked, API Connect requires https to be used,
	* In the `Base Path` section enter `/`,
	* In the `Consumes` section, check the `application/json` value,
	* In the `Produces` section, check the `application/json` value,
	* In the Paths section, change the `/ping` endpoint to the Loopback generated `/getPing` endpoint,
	* Save the Draft API,


4. Add Assembly to API

	* In the API Designer, in the API Draft, go to the `Assemble` tab,
	* Or in the API Designer go to the `Policy Assembly` section, 
	* Click the `Create assembly` button,
	* You should be in the `Assemble` designer,

		![Assemble Designer](../images/apic-assemble-designer.png)

	* Make sure `DataPower Gateway policies` is selected,
	* Under `Logic`, drag the `Switch` action into the assembly,
    	* Configure the `Switch` action, and for `case 0`, `search operations` and select the `GET /getPing` operation,
    	* Click the `+ Otherwise` button,
  	* Save the Draft API,

		![Assemble switch action](../images/apic-assemble-switch.png)

  	* Loopback created the following endpoints,

		![Loopback Endpoints](../images/loopback-endpoints.png)

	* As you see, Loopback 3 prefixes the methods by the Model name. In the assemble, we therefor need to set a variable `lbmodel` to be used as a prefix in the URI of the proxy URL.
    	* Under `Policies`, drag the `Set Variable` into the switch flow for `GET /getPing`,
    	* Click the `+ Action` button,
    	* Under `Set` define the name of the variable as `lbmodel`,
    	* Under `Value` define the value of the model as `Pings`,
		* Save the Draft API,

			![Loopback Assemble Designer Set Variable 1](../images/apic-assemble-designer-set-variable-1.png)

		* Under `Policies`, drag another `Set Variable` into the switch flow for `otherwise`,
		* Click the `+ Action` button,
    	* Under `Set` define the name of the variable as `lbmodel`,
    	* Under `Value` define the value of the model as `Messages`,
    	* Save the Draft API,

			![Loopback Assemble Designer Set Variable 2](../images/apic-assemble-designer-set-variable-2.png)

	* Under `Policies`, drag the `Proxy` action onto the assembly, after the `switch` logic node but before the endnode to capture both the case0 and otherwise workflow,
    	* Configure the `Proxy` action, and add the URL for the Guestbook API service on Kubernetes appended by the contaxt variable `$(request.path)` representing the URI of the request, e.g. `http://remkohdev-cluster-3w.us-south.containers.appdomain.cloud:31356/api/$(lbmodel)$(request.path)`,
    	* Note that the `request.path` includes the starting `/`,
    	* Save the Draft API,

			![APIConnect Assemble Designer Proxy](../images/apic-assemble-designer-proxy.png)

	* Test the Draft API, click the `play` icon to test,
	* A `Setup` window will open on the left side,

		![Assemble Test Setup](../images/apic-assemble-test-setup.png)

	* Because we do not have a product yet, in the `Or create a new product and publish it to the selected catalog` section, enter a name `Guestbook API`, if you already created the product you can select the product from the `Choose an existing product` dropdown,
	* Click the `Create and publish` button,
	* Click `Next`
	* Click the `Republish product` button,
	* In the `Operation` section, for `Operation` select the `GET /getPing` operation,
	* Click the `Invoke` button

5. Add Draft to Product

	* During testing using the `Invoke` option, you already created a product,
	* If you do not have a product already, you can generate a product from the top right dropdown, select `Generate a default product`,
    	* Change the title to `Guestbook API`,
    	* Click the `Create product`,
  	* Otherwise, select the `Add to existing products`,
    	* Select the appropriate product version,
    	* Click `Add`,
  	* Click `All APIs` and go to `Products`,
  	* Click the `Guestbook API 1.0.0` to edit the product settings,
	* For now, keep the default product settings,
	* From the `Home` dropdown menu, select `Dashboard`,
	* From the product instance dropdown menu to the right of the product, you can manage the lifecycle of the product,
