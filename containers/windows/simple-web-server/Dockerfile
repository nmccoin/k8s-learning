# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore/iis

# Metadata indicating an image maintainer.
LABEL maintainer="daniel.slater@providence.org"

# remove unneeded files

RUN powershell -NoProfile -Command Remove-Item -Recurse C:\inetpub\wwwroot\*

RUN powershell -NoProfile -Command remove-webbinding -protocol http

#Set working directory
WORKDIR /inetpub/wwwroot
 
# expose port to container
EXPOSE 9000

# Creates an HTML file and adds content to this file. 
COPY wwwroot .

### Didn't get this section working, hacked together via commandline and then committed image

#RUN powershell -Command new-webbinding -protocol http -port 9000 -name "Default Web Site"
