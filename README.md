# Jenkins-Sonarqube-Docker

1. Infrastructure Setup (AWS EC2)
 We need 3 servers
1. Jenkins Server
* Install Jenkins
* Install required plugins
2. SonarQube Server
* Install SonarQube (needs separate non-root user + Java 17)
* Expose on port 9000
3.Docker Server
* Docker installed
* Used for hosting the final container
* Allows 8085 port 
2. Install Jenkins
  On Jenkins EC2 instance:
  ```
  sudo apt update
  sudo apt install openjdk-17-jdk -y
  wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt update
  sudo apt install jenkins -y
  sudo systemctl enable jenkins
  sudo systemctl start jenkins
  
  ```
  Open Jenkins → http://<Jenkins-EC2-IP>:8080

3. Install Docker on Docker Servers
   On Docker EC2 instance:
 ``` 
  sudo apt update
  sudo apt install docker.io -y
  sudo systemctl start docker
  sudo systemctl enable docker
  sudo usermod -aG docker ubuntu
 ```
4. Install SonarQube
   On Sonarqube EC2 instance:
 ```
  sudo adduser sonar
  sudo su - sonar
  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.4.87374.zip
  unzip sonarqube-9.9.4.87374.zip
  cd sonarqube-9.9.4.87374/bin/linux-x86-64
  ./sonar.sh start

 ```
   Access SonarQube → http://<SonarQube-EC2-IP>:9000
   Default login: admin/admin
   Generate a SonarQube Token for Jenkins.
5.  Jenkins Plugins
  * Install in Jenkins:
  * GitHub Integration
  * SonarQube Scanner for Jenkins
  * SSH2  
  * Docker
6. Setup Jenkins Pipeline (Freestyle Job)

1. Create New Job
* Open Jenkins Dashboard → Click “New Item”
* Enter job name Automation-Pipeline
* Select Freestyle project → Click OK
2. Source Code Management
* Select Git
* Enter your repository URL.
* Specify branch to build.
3. Build Triggers
   Poll SCM → Schedule:
  ```
  * * * * *
  ```
 
4. Environment
 * Delete workspace before build starts

5. Build steps → Exceute shell Command
```
ssh -o StrictHostKeyChecking=no root@3.91.244.124 "rm -rf ~/website/*"
scp -r /var/lib/jenkins/workspace/Automated-Pipeline/* root@3.91.244.124:/home/ubuntu/website

ssh root@3.91.244.124 "cd /home/ubuntu/website && docker build -t mywebsite . && docker run -d -p 8085:80 --name=Gamewebsite mywebsite"

```
6. Save & Build
* Click Save
* Run Build Now to test
* Check logs in Console Output
7. Verify Deployment
In Browser:
```
http://Docker_SERVER_IP>:8085
```
7.  Pipeline Flow
* Developer pushes code to GitHub → Github triggers Jenkins.
* Jenkins pulls code → runs SonarQube scan.
* If code passes → Jenkins builds Docker image.
* Jenkins connects via SSH to deployment EC2 → pulls image → runs container.
* Website available at: http://<Deployment-EC2-IP>:8085
     
