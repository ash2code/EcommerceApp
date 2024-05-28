# Use a lightweight Tomcat image with Java 17
FROM tomcat:latest


RUN apt-get update && apt-get install -y maven

# Create a directory for the application
WORKDIR /opt/app

# Copy the Maven project directory
COPY . .

# Install dependencies using Maven (assuming a Maven wrapper is present)
RUN mvn clean install

# Copy the WAR file to the Tomcat deployment directory
COPY target/*.war /usr/local/tomcat/webapps/

# Expose the default Tomcat port
EXPOSE 8080

# Set the default command (optional)
CMD ["catalina.sh", "run"]
