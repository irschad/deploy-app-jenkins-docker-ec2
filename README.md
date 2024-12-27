# Deploy Application from Jenkins Pipeline to EC2 Instance (using Docker)

## Project Overview
This project demonstrates the deployment of a web application to an AWS EC2 instance directly from a Jenkins pipeline using Docker. The process automates Continuous Deployment (CD) by extending a CI pipeline with a deployment stage.

---

## Technologies: **AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub**

---

## Project Description
The project involves the following steps:
1. **Prepare the AWS EC2 Instance for Deployment**: Install Docker on the instance.
2. **Set Up Jenkins Credentials for SSH Access**: Create SSH key credentials for the EC2 server.
3. **Extend CI Pipeline with Deployment Step**:
   - Connect to the EC2 instance from Jenkins via SSH.
   - Deploy the Docker image built in the Jenkins CI pipeline.
4. **Configure Security Group for EC2**: Open necessary ports to access the web application.

---

## Steps to Prepare AWS EC2 Instance for Deployment

### Install Docker
1. Log in to your EC2 instance via SSH.
2. Install Docker using the following commands:
   ```bash
   sudo yum update -y
   sudo yum install docker -y
   sudo service docker start
   sudo usermod -aG docker ec2-user
   ```
3. Verify the Docker installation:
   ```bash
   docker --version
   ```

---

## Steps to Create SSH Key Credentials in Jenkins
1. Install the **SSH Agent** plugin:
   - Navigate to **Manage Jenkins** > **Manage Plugins**.
   - Search for and install the "SSH Agent" plugin.
2. Add SSH Credentials:
   - Go to **Dashboard** > Your Project > **Credentials**.
   - Click on **Global credentials (unrestricted)** > **Add Credentials**.
   - Select **SSH Username with private key**.
   - Provide:
     - **ID**: `ec2-server-key`
     - **Username**: `ec2-user`
     - **Private Key**: Paste the private key contents.
   - Save the credentials.
3. Update EC2 Security Group:
   - Add the Jenkins server IP to the EC2 instance's inbound rules for port `22` (SSH).

---

## Steps to Extend CI Pipeline with Deployment Stage

### Add Deployment Stage to Jenkinsfile
 Edit the `Jenkinsfile` in the project repository to include a deployment stage:
   ```groovy
   stage('Deploy Application') {
       steps {
           script {
               echo 'Deploying Docker image to EC2 server...'
               def dockerCmd = 'docker run -d -p 8080:8080 irschad/java-app:4.0'
               sshagent(['ec2-server-key']) {
                   sh "ssh -o StrictHostKeyChecking=no ec2-user@34.229.224.203 ${dockerCmd}"
               }
           }
       }
   }
   ```

### Log In to Docker Hub on EC2 instance
1. SSH into the EC2 instance:
   ```bash
   ssh -i ~/.ssh/ec2-server-key.pem ec2-user@34.229.224.203
   ```

2. Log in to Docker Hub:
   ```bash
   docker login
   ```

---

## Configure Security Group for Web Application Access
1. Open the EC2 instance's security group settings in the AWS console.
2. Add an inbound rule to allow access to port `8080`:
   - **Type**: Custom TCP
   - **Port Range**: 8080
   - **Source**: Anywhere (0.0.0.0/0)

---

## Deploy and Access the Web Application
1. Trigger the Jenkins pipeline to build and deploy the application automatically via webhook.
2. Check whether the container is running on the target instance:
   ```bash
   docker ps 
   CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                                       NAMES
   b91cb1618412   irschad/java-app:4.0   "java -jar java-mave…"   9 seconds ago   Up 8 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   confident_li
   ```
   
3. Access the deployed application in a browser using the URL:
   ```
   http://34.229.224.203:8080
   ```
  ![Java Maven App](https://github.com/user-attachments/assets/1d3f4589-7c85-40bc-912b-f78e5adc3a57)


## Notes
- Ensure the Jenkins server can connect to the target EC2 instance via SSH.
- Update the security groups responsibly to prevent exposing unnecessary ports.


