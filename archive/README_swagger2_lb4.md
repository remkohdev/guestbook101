# Lab0 - Create the Guestbook API App using Loopback

1. Pre-requirements

	* Loopback 3 CLI
	* APIC CLI
	* IBM Cloud CLI
	* Bash

2. Setup
   
	* Create the project directory

		```console
		$ mkdir guestbook101
		$ cd guestbook101
		```

3. Get the Guestbook UI source code

	```console
	$ git clone https://github.com/IBM/guestbook.git
	$ cd guestbook/v2/guestbook
	$ docker stop guestbook-web
	$ docker rm guestbook-web
	$ docker build --no-cache -t guestbook-web .
	$ docker run -d --restart always --name guestbook-web -p 3000:3000 guestbook-web
	$ cd ../../..
	```

4. Create the Guestbook API

	* Create the Loopback 4 project for the Guestbook API, and run the app on port 3001, because the UI is already running on port 3000,

		```console
		$ lb guestbook-api
		$ cd guestbook-api
		$ PORT=3001 npm start
		```

	* Add a Dockerfile to your application, 

		```text
		
		```

	* Run the app with Docker,

		```console
		$ docker stop guestbook-api
		$ docker rm guestbook-api
		$ docker build --no-cache -t guestbook-api .
		$ docker run -d --restart always --name guestbook-api -p 3001:3001 guestbook-api
		```

5. Create the OpenAPI spec using version 2,

	* Create a default OpenAPI spec as follows,

		```console
		$ mkdir api
		$ vi api/guestbook-api-openapi-v2-0.yaml
		```

	* And past the following spec,

		```yaml
		swagger: "2.0"
		info:
		  title: Guestbook API
		  description: Guestbook API provides an API for the Guestbook application
		  version: 1.0.0
		basePath: /v1.0
		schemes:
		  - https
		paths:
		definitions:
		```

	* Next the data model can be represented in the `definitions` section,

		```yaml
		definitions:
		  Message:
		    title: Message
			properties:
			  id:
			    type: number
			  text:
				type: string
			  datecreated:
				type: string
				format: date-time
			  personId:
				type: number
			required:
			  - text
		  Person:
			title: Person
			properties:
			  id:
				type: number
			  firstname:
				type: string
			  lastname:
				type: string
			  username:
				type: string
		  Contact:
			title: Contact
			properties:
			  id:
				type: number
			  email:
				type: string
			  phone:
				type: string
			  personId:
				type: number
		  Address:
			title: Address
			properties:
			  id:
				type: number
			  addressline1:
				type: string
			  addressline2:
				type: string
			  city:
				type: string
			  state:
				type: string
			  zipcode:
				type: string
			  country:
				type: string
			  contactId:
				type: number
		  Ping:
		    title: Ping
		    properties:
			  greeting:
				type: string
			  date:
				type: string
			  url:
				type: string
			  headers:
			 	type: object
				properties:
				  Content-Type:
				    type: string
		```
	* Add the following endpoints,
    	* Health:
        	* GET /ping
    	* Messages:
        	* GET /messages
        	* POST /messages

	* Add `GET /ping`

		```yaml
		  /ping:
			get:
			summary: Does a ping check to see if the server is up
			responses:
				'200':
				description: Ping Response
				schema:
					$ref: '#/definitions/Ping'
			operationId: PingController.ping
	  	```

	* Add Messages endpoints. For simplicity, add 2 endpoints to find and create messages in the Guestbook, a `GET /messages` and a `POST /messages`,

		```yaml
		paths:
		  /messages:
			get:
			  tags:
			    - Messages
			  parameters:
				- name: filter
				  in: query
				  type: array
				  items:
					type: string
				- name: offset
				  in: query
				  type: integer
				  minimum: 0
				- name: limit
				  in: query
				  type: integer
				  minimum: 0
			responses:
			  '200':
				description: Array of Message model instances
				schema:
				  type: array
				  items:
					$ref: '#/definitions/Message'
			operationId: MessagesController.find
			post:
			tags:
				- Messages
			parameters:
				- name: message
				description: new Message
				in: body
				schema:
					type: object
					$ref: '#/definitions/Message'
			responses:
				'201':
				description: Message model instance
				schema:
					$ref: '#/definitions/Message'
			operationId: MessagesController.create
		```

	* Add API Connect context variables,

		* You can add context variables for the IBM DataPower Gateway in API Connect by using the [IBM extensions to the OpenAPI (Swagger 2.0) specification](https://www.ibm.com/support/knowledgecenter/en/SSFS6T/com.ibm.apic.toolkit.doc/rapim_cli_swagger_extensions.html) with `x-ibm-configuration`. To define properties in an API use the [`properties` extension](https://www.ibm.com/support/knowledgecenter/SSFS6T/com.ibm.apic.toolkit.doc/rapim_cli_properties.html),

			```yaml
			x-ibm-configuration:
				properties:
					guestbook_svc_url:
						value: 'http://69.63.218.104:32145'
						description: Location of the Guestbook API service
						encoded: false
			```

		* To use the `guestbook-cvs-url` in the `Invoke` action of an assembly, you use the templating syntax `$()`. To inject the `guestbook_svc_url` as the URL property of the `Invoke` action in the assembly, use `http://$(guestbook_svc_url)/$(request.uri)`.

6. Generate the Loopback application artifacts from the OpenAPI spec version 2.0,

	```console
	$ lb4 openapi api/guestbook-api-openapi-v2-0.yaml --validate
	Loading api/guestbook-api-openapi-v2-0.yaml...
	? Select controllers to be generated: (Press <space> to select, <a> to toggle all, <i> to invert selection)[Messages] MessagesController, [Health] HealthController
	  create src/models/message.model.ts
	  create src/models/person.model.ts
	  create src/models/contact.model.ts
	  create src/models/address.model.ts
	  create src/models/ping.model.ts
	  create src/controllers/messages.controller.ts
	  create src/controllers/health.controller.ts
	```


7. Create a DataSource,
   
    * To add relations to models as Persistent Entity, you must first create a repository for the models. But to create a repository for a Persistent Entity, you must first create a DataSource.
    * For now, I will create a simple in-memory DataSource,

		```console
		$ lb4 datasource
		? Datasource name: db
		? Select the connector for db: In-memory db (supported by StrongLoop)
		? window.localStorage key to use for persistence (browser only): 
		? Full path to file for persistence (server only): 
		  create src/datasources/db.datasource.json
		  create src/datasources/db.datasource.ts
		  update src/datasources/index.ts

		Datasource Db was created in src/datasources/
		```

	* I did not define file and full path for persistence, but add a file name to persist the `in-memory' data,
   
7. Add `in-memory` Repositories,

	* Now you have a DataSource, you can create the Repository.
	* Select datasource `DbDatasource`, select all 4 data models: Address, Contact, Message, Person but leave the  Ping unchecked, and select the `DefaultCrudRepository` as repository base class,
  
		```console
		$ lb4 repository
		? Please select the datasource DbDatasource
		? Select the model(s) you want to generate a repository Address, Contact, Message, Person
		? Please select the repository base class DefaultCrudRepository (Legacy juggler bridge)
		  create src/repositories/address.repository.ts
		  create src/repositories/contact.repository.ts
		  create src/repositories/message.repository.ts
		  create src/repositories/person.repository.ts
		  update src/repositories/index.ts
		  update src/repositories/index.ts
		  update src/repositories/index.ts
		  update src/repositories/index.ts

		Repositories AddressRepository, ContactRepository, MessageRepository, PersonRepository were created in src/repositories/
		```

8. Add Implementation to the Controller,

	* Edit the file `src/controllers/messages.controller.ts`,

		```javascript
		
		```

9.  Run the new Application, 

    * Run your application as a container,

		```console
		docker stop guestbook-api
		docker rm guestbook-api
		docker build --no-cache -t guestbook-api .
		docker run -d --restart always --name guestbook-api -p 3001:3001 guestbook-api
		```

	* Visit the /explorer to see the created endpoints,

		![Guestbook API](../images/guestbook-api-explorer.png)

8.  Test
   
   ```console
   $ curl -X POST "http://localhost:3001/messages" -H "accept: application/json" -H "Content-Type: application/json" -d "{\"text\":\"hello1\",\"datecreated\":\"2019-09-09T15:39:35.357Z\"}"
   ```

11. OpenAPI spec,

	* The OpenAPI spec is available at:
    	* In JSON format: http://localhost:3000/openapi.json,
    	* In YAML format: http://localhost:3000/openapi.yaml, 


