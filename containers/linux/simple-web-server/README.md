# Simple Web Server Container

This container is configured to listen on port 9000 and serve up a static website.

## Modifying the container and website

### Website
All files for the website are stored in the '[public-html](public-html/)' folder. These files are copied to the htdocs directory inside
the container during container build.

### Container
The [Dockerfile](Dockerfile) can be modified to change the uuid or guid of the user running the httpd daemon by changes these values
* usermod -u `<150>` www-data
* groupmod -g `<150>` www-data

If you wish to change the port the container is listening on, edit the [my-httpd.conf](my-httpd.conf) file and change the following line
* Listen <9000>

### Building container
After modifying any of the information above, you will have to rebuild the container on your local workstation.
This is only an example of the command.
* docker build -t <imagename> .

The image will still need to be tagged and pushed to your ACR, as well as updating the deployment files to leverage
the new container.
