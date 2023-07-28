
# Setting up Declarative Jenkins and CI/CD Pipeline for Django Notes App

This documentation provides step-by-step instructions on how to set up Jenkins, Docker, and a CI/CD pipeline to build, push, and deploy the Django Notes App using GitHub webhooks.




## Prerequisites
Before proceeding with the setup, ensure you have the following prerequisites:

1. An AWS EC2 instance with Ubuntu OS.
2. A security group allowing inbound traffic on ports 22 (SSH) and 8080 (Jenkins).
3. A key pair to SSH into the EC2 instance.
4. An AWS IAM user with necessary permissions for Jenkins.
5. Docker installed on the EC2 instance.


## Steps
### 1. Create an EC2 instance for Jenkins
1. Launch an AWS EC2 instance with Ubuntu OS.
2. Set up security groups to allow inbound traffic on ports 22 and 8080.
### 2. SSH into the EC2 instance
```bash
ssh -i /path/to/your/key.pem ubuntu@your_ec2_public_ip
```
### 3. Install Docker on the EC2 instance
```bash
sudo apt-get update
sudo apt-get install docker.io
sudo usermod -aG docker ubuntu
sudo reboot
```
### 4. Clone the Django Notes App from GitHub
```bash
git clone https://github.com/LondheShubham153/django-notes-app.git
```
### 5. Create a Dockerfile for the Django App
Create a Dockerfile in the root directory of the Django Notes App with the following contents:
```bash
# Start with the official Python 3.9 image from Docker Hub as the base image.
FROM python:3.9

# Set the working directory inside the container to /app/backend.
WORKDIR /app/backend

# Copy the 'requirements.txt' file from the host machine to the '/app/backend' directory inside the container.
COPY requirements.txt /app/backend

# Install the Python dependencies listed in the 'requirements.txt' file using pip inside the container.
RUN pip install -r requirements.txt

# Copy the entire current directory (where the Dockerfile is located) to the '/app/backend' directory inside the container.
COPY . /app/backend

# Expose port 8000 to the outside world. This allows external communication with the Django server running on port 8000 inside the container.
EXPOSE 8000

# Set the default command to run when the container starts.
# In this case, it runs the Django development server with 'manage.py runserver' on 0.0.0.0:8000, which means it will listen on all available network interfaces.
CMD python /app/backend/manage.py runserver 0.0.0.0:8000
```
### 6. Build the Docker Image
```bash
docker build -t notes-app .
```
### 7. Install Jenkins on the EC2 instance
```bash
sudo apt install openjdk-17-jre
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install Jenkins
```
### 8. Start Jenkins and Set Up Initial Configuration
```bash
sudo service jenkins start
sudo service jenkins status
sudo usermod -aG docker jenkins
```
### 9. Access Jenkins Dashboard
Navigate to your EC2 instance's public IP on port 8080 in your web browser.

### 10. Set Up Jenkins Global Credentials
1. Go to "Manage Jenkins" > "Manage Credentials".
2. Click "Global credentials (unrestricted)" > "Add Credentials".
3. Add your Docker Hub credentials (username and password) as global credentials to be used in the pipeline.

### 11. Create a Jenkins Pipeline
1. Click "New Item" to create a new pipeline job.
2. Name the pipeline job (e.g., "Notes-django-app-CICD-pipeline") and choose "Pipeline" as the type.
3. Under the "Pipeline" section, select "Pipeline script from SCM" for "Definition".
4. Use Jenkinsfile where the declararive pipeline script is written.
5. Choose "Git" as the SCM and provide your Django Notes App repository URL.
6. Set the branch to "main".
7. Save the pipeline job.

### 12. Configure Docker Compose
Install Docker Compose on the EC2 instance:
```bash
sudo apt-get install docker-compose
```
Create a docker-compose.yml file in the root directory of the Django Notes App with the following contents:
```bash
version: "3.3"
services:
  web:
    image: monseiury/notes-app:latest
    ports:
      - "8000:8000"
```

### 13. Set Up GitHub Webhook
1. Go to your Django Notes App repository on GitHub.
2. Click on "Settings" > "Webhooks" > "Add webhook".
3. In the "Payload URL" field, enter http://your_ec2_public_ip:8080/github-webhook/.
4. Select "Content type" as application/json.
5. Set up a secret if needed.
6. Under "Which events would you like to trigger this webhook?", choose "Just the push event" or specific events.
7. Click "Add webhook" to save the configuration.

### 14. Test the CI/CD Pipeline
Make changes to your Django Notes App code and push to GitHub. The webhook will trigger the Jenkins pipeline, which will build the Docker image, push it to Docker Hub, and deploy the app using Docker Compose on port 8000.

### 15. Access the Deployed App
Navigate to http://your_ec2_public_ip:8000/ in your web browser to access the deployed Django Notes App.

## Conclusion
Congratulations! You have successfully set up Jenkins, Docker, and a CI/CD pipeline to build, push, and deploy the Django Notes App automatically whenever changes are pushed to GitHub.
