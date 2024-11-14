# SBA-API-LINUX

To deploy a Spring Boot JAR file on a Linux server with Apache HTTP Server, you can follow these steps. Since Apache HTTP Server can’t directly run Java applications, we’ll set up the Spring Boot application as a standalone service and configure Apache as a reverse proxy to forward requests to it. Here’s a guide on how to do this:

1. Transfer the JAR File to the Server

	1.	Build the JAR:
	•	Package the Spring Boot application as a JAR file using Maven or Gradle:

./mvnw clean package  # or `mvn clean package`


	2.	Transfer the JAR to the Linux Server:
	•	Use scp or another file transfer tool:

scp target/your-app.jar user@your-server-ip:/path/to/deployment/folder



2. Install Java on the Server

Ensure that the correct version of Java is installed on the server (use the same version as your Spring Boot application):

sudo apt update
sudo apt install openjdk-17-jdk -y  # Install the required version of Java
java -version  # Verify the installation

3. Run the Spring Boot Application as a Service

To make sure your Spring Boot application runs reliably, even after server reboots, you can set it up as a systemd service.
	1.	Create a Systemd Service File:
	•	Create a new file in /etc/systemd/system/ with a .service extension. For example:

sudo nano /etc/systemd/system/your-app.service


	2.	Add the Service Configuration:
	•	Replace placeholders like your-app, /path/to/deployment/folder, and java -jar your-app.jar with the actual paths and commands for your application.

[Unit]
Description=Your Spring Boot Application
After=syslog.target

[Service]
User=your-username  # Replace with the user who should run the app
ExecStart=/usr/bin/java -jar /path/to/deployment/folder/your-app.jar
SuccessExitStatus=143
Restart=always
RestartSec=10
StandardOutput=file:/var/log/your-app/output.log
StandardError=file:/var/log/your-app/error.log

[Install]
WantedBy=multi-user.target


	•	Make sure to replace /path/to/deployment/folder/your-app.jar with the full path to your JAR file.

	3.	Start and Enable the Service:

sudo systemctl daemon-reload
sudo systemctl start your-app
sudo systemctl enable your-app


	4.	Verify the Service:
	•	Check that the service is running:

sudo systemctl status your-app


	•	View logs if needed:

tail -f /var/log/your-app/output.log



4. Set Up Apache as a Reverse Proxy

If you want to make your Spring Boot application accessible on port 80 (standard HTTP) or 443 (HTTPS) via Apache, set up Apache as a reverse proxy.
	1.	Install Apache:

sudo apt update
sudo apt install apache2 -y


	2.	Enable Proxy Modules:
	•	Enable the necessary Apache modules for proxying:

sudo a2enmod proxy
sudo a2enmod proxy_http


	3.	Configure Apache for Reverse Proxy:
	•	Create or edit an Apache configuration file, typically in /etc/apache2/sites-available/.

sudo nano /etc/apache2/sites-available/your-app.conf


	•	Add the following configuration, where localhost:8080 is the port your Spring Boot application runs on:

<VirtualHost *:80>
    ServerName your-domain.com

    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/

    ErrorLog ${APACHE_LOG_DIR}/your-app-error.log
    CustomLog ${APACHE_LOG_DIR}/your-app-access.log combined
</VirtualHost>


	•	Replace your-domain.com with your domain name or server IP.

	4.	Enable the Site and Restart Apache:

sudo a2ensite your-app.conf
sudo systemctl restart apache2



5. (Optional) Set Up SSL with Let’s Encrypt

To secure your site with HTTPS, use Let’s Encrypt.
	1.	Install Certbot:

sudo apt install certbot python3-certbot-apache -y


	2.	Generate an SSL Certificate:

sudo certbot --apache -d your-domain.com


	3.	Follow the Prompts to complete the SSL setup, and Certbot will automatically configure Apache to redirect HTTP traffic to HTTPS.

Summary

	1.	Transfer and run the JAR as a service for reliability.
	2.	Configure Apache as a reverse proxy to forward requests to the Spring Boot application.
	3.	(Optional) Enable SSL with Let’s Encrypt for secure HTTPS access.

This setup makes your Spring Boot application available on a standard web port, providing reliability and security through systemd and Apache.
