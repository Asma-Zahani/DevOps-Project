FROM tomcat:10.1-jdk17

# Copy the generated WAR into Tomcat's webapps as ROOT.war
# The build produces a WAR (packaging=war) so we deploy it to Tomcat.
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

# Note: the container will serve the app on port 8080 inside the container.
# Use the compose file to map it to the host (Jenkins will build images and push to Harbor).
