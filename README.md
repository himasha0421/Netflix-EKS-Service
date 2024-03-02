# Netflix-EKS-Service
Netflix clone deployment inside AWS EKS cluster

# Stage 1: Development

### step 1. setup a development server 

create server with good amount of resources , because we are going to run jenkins, sonarqube services ontop this server.

### setp 2. clone the git repo

```bash
git clone https://github.com/himasha0421/Netflix-EKS-Service.git
```

### Step 3: Get the API Key

* Open a web browser and navigate to TMDB (The Movie Database) website.
* Click on "Login" and create an account.
* Once logged in, go to your profile and select "Settings."
* Click on "API" from the left-side panel.
* Create a new API key by clicking "Create" and accepting the terms and conditions.
* Provide the required basic details and click "Submit."
* You will receive your TMDB API key.

### Step 4: Install Docker and Run the App Using a Container

* setup docker on EC2 instance

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
* Build and run your application using Docker containers

```bash
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

#to delete
docker stop <containerid>
docker rmi -f netflix
```


# Stage 2: Security

1. Install SonarQube and Trivy:

* Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

> Sonarqube

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

> Trivy

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy        
```

to scan image using trivy

```bash
trivy image <imageid>
```

# Stage 3: Ops (CI/CD Setup)

## 1. Install Jenkins for Automation:

* Install Jenkins on the EC2 instance to automate deployment: 

> Install Java

```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

> Install Jenkins

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## 2. Install Necessary Plugins in Jenkins:

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1. Eclipse Temurin Installer (Install without restart)

2. SonarQube Scanner (Install without restart)

3. NodeJs Plugin (Install Without restart)

4. Email Extension Plugin

Configure Java and Nodejs in Global Tool Configuration

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

## 3. SonarQube Configuration

* Create the token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

After adding sonar token

Click on Apply and Save

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.
