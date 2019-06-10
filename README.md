
# RES-2019-Labo-HTTPInfra

Link to Teaching Repo : https://github.com/SoftEng-HEIGVD/Teaching-HEIGVD-RES-2019-Labo-HTTPInfra

## Step 1: Static HTTP server with apache httpd

To access our webpage we need to first build the image from our Dockerfile, run a container using that image and then finally access the web content. The Dockerfile for this server simply specifies that we want to use the official apache server image as a base (FROM) and that we want the container to copy (COPY) the content/ folder to its own /var/www/html/ folder. That folder will allow the server to display the webpage index.html.

After cloning the repo, build the image and run a container using the following commands while in the docker-images/apache-php-image/ folder :

```dockerRun
$ docker build -t res/apache_php .
$ docker run -d -p 9090:80 res/apache_php
```

The page is now accessible through the browser by going to Docker-machine's IP address and specifying the port, which is most likely this address : 192.168.99.100:9090

## Step 2: Dynamic HTTP server with express.js

After creating a static web page we now want to create a dynamic web page using node.js and the chance module (to generate random values for our content). This server will be able to reply to requests by sending a .json payload.

Same as before, we used a Dockerfile coupled with copying the src/ folder found in the docker-images/express-image/ folder into the container to create our desired server.

```dockerRun
$ docker build -t res/express_students .
$ docker run -d -p 9090:3000 res/express_students
```

By going to Docker-machine's IP address and specifying the port 9090 this time (which we mapped to the container 3000 port with the run command), we'll display a randomly generated .json package every time we refresh.

## Step 3: Reverse proxy with apache (static configuration)

Initially we run these 4 docker commands to start up the required containers.

```dockerRun
$ docker run -d --name apache_static res/apache_php
$ docker run -d --name express_dynamic res/express_movies
$ docker build -t res/apache_rp .
$ docker run -d --name apache_rp res/apache_rp
```

Assuming the staticly assigned IPs have not changed you should be ready to access the two sites via the reverse proxy.

Otherwise they will have to be changed in this file *001-reverse-proxy.conf*

The ip addresses of the container can be found using this command

*inspect docker inspect <container_name> | grep -i ipaddress*

**Warning :** You cannont reach the reverse proxy using the IP address.

This is beacause our default configuration in *000-default.conf* is empty to ensure you access the sites through the url.

To access the server via a browser we have to add a line in our DNS configuration.

Access the file with the following command

```bash
$ sudo nano /etc/hosts
```

and add *172.17.0.4	demo.res.ch* to the file.

This is a stopgap solution as the ip should be added dynamically since it may differ depending on the order we activated the containers and if any were running before.

To test the proxy all you need to do is launch a browser and type *demo.res.ch:8080* in the search bar to access the *apache_static* content.

Type *demo.res.ch:8080/api/students/* to access the *express_dynamic* content.

## Step 4: AJAX requests with JQuery

For this step we "created" a javascript file called students.js

```javascript
$(function() {
	console.log("loading students");
	
	function loadStudents() {
		$.getJSON( "/api/studens/", function( students ) {
			console.log(students);
			var message = "Nobody is here";
			if( students.length > 0 ) {
				message = students[0].firstName + " " + students[0].lastName;
			}
			$(".skills").text(message);
		});
	};
	
	loadStudents();
	setInterval( loadStudents, 2000 );
});
```

and added two lines to the *index.html* file.

The first to add our custom script and the seccond, added in the intro content, to display the result of the ajax request.

```html
<script src="js/students.js"></script>

<span class="skills">HTTP fluff text!</span>
```

To test this simply get to the *apache_static* content the same way as in step 3 and now you can a randomly generated name under the text *Le cours de RES c'est fun!*.

To prove the request is dynamic, the script requests a new student list every 2 secconds.

As you can see the name chages without having to reload the whole page.

## Step 5: Dynamic reverse proxy configuration

*placeholder*

## Additional Steps

### Management UI

For the management UI, we chose to use [Portainer](https://portainer.io/) as we found it to be the easiest to implement.

To run Portainer, use the following commands : 

```bash
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

You should now be able to access the docker container via your browser on port 9000.
