# CICD Engineering assessment

# Tools used
1. jenkins
2. docker
3. sonarqube
4. Maven
5. CICD



# A visual representation of the solution 
<img src="images/Diagram.png" alt="Alt text">



# steps
1. spin up an instances in AWS
2. connect via ssh
3. configure the instance
4. installation of jenkins
5. installation of sonaqube
6. installation of maven
7. installation of docker


# Installing Jenkins
Add Jenkins Repository Key

    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    
Add Jenkins Repository

    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    
Update Package Lists

    sudo apt update
    
Install Jenkins

    sudo apt install jenkins
    
Start and Enable Jenkins

    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    
# connect to jenkins
open a web browser and go to http://localhost:8080 or http://your_server_ip:8080.

You will be prompted to enter the initial admin password. Retrieve it with:


    sudo cat /var/lib/jenkins/secrets/initialAdminPassword


install the required plugins - maven Docker, sonaqube scanner
and add the necessary credentials


# installation of docker
Update the Package Lists

    sudo apt update
    
Install Docker

    sudo apt install docker.io
    
Start and Enable Docker


    sudo systemctl start docker
    sudo systemctl enable docker
    
Verify Docker Installation

    docker --version

# run sonarqube as a container because the installation process is complex

    docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
# connect to sonarqube
open a web browser and go to http://localhost:9000 or http://your_server_ip:9000

## CICD Pipeline
CI/CD stands for Continuous Integration and Continuous Deployment (or Continuous Delivery). It's a set of best practices, principles, and automated tools used in software development to ensure that code changes are tested and deployed efficiently and reliably.

Here's a brief overview of CI/CD:

Continuous Integration (CI):

Integration: Developers regularly merge their code changes into a shared repository (version control system like Git).
Automated Testing: Automated tests are run to validate that the new code doesn't break existing functionality.
Early Detection of Issues: CI helps catch and fix integration issues early in the development process.
Continuous Deployment (CD):

Deployment Automation: Once code changes pass the CI phase, they are automatically deployed to production or staging environments.
Release Management: This can include processes for managing versioning, release notes, and rollback strategies.
Continuous Delivery (CD):

Similar to Continuous Deployment: The main difference is that with Continuous Delivery, code changes are automatically deployed to staging or pre-production environments, but deployment to production is done manually.
Benefits:

## CICD process
Jenkins was used to setup the CICD pipeline using pipeline as code method (groovy), also jenkins shared library was also intergrated to make the code flexible and re-useable [Jekins Shared Library repo link](https://github.com/Hybeekay1/Jenkins_Shared_Library.git)

# jenkins pipelin code


        @Library('my-shared-library') _
        pipeline {
           agent any
           parameters{
                choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
                string(name: 'ImageName', description: "name of the docker image", defaultValue: 'pet-clinic')
                string(name: 'HubUser', description: "Docker Hub user", defaultValue: 'malik0x')
           }
           stages {
            stage('Git Checkout'){
            when { expression {  params.action == 'create' } }
              steps {
                 gitCheckout(
                    branch: "main",
                    url: "https://github.com/Hybeekay1/SpringPet-Clinic.git"
                 )
                    }
                }
              stage('Maven Build'){
              when { expression {  params.action == 'create' } }
              steps {
                 mvnBuild()
                    }
                }
              stage('Testing'){
              when { expression {  params.action == 'create' } }
              steps {
                 script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    codeTest(SonarQubecredentialsId)
                       }
                    }
                }
              stage('Docker Build'){
              when { expression {  params.action == 'create' } }
              steps {
                 script {
                    dockerBuild("${params.ImageName}","${params.HubUser}")
                       }
                    }
                }
              stage('Docker Push To Docker Hub'){
              when { expression {  params.action == 'create' } }
              steps {
                 script {
                    dockerImagePush("${params.ImageName}","${params.HubUser}")
                       }
                    }
                }
              stage('App Deploy'){
              when { expression {  params.action == 'create' } }
              steps {
                 script {
                    deployApp("${params.ImageName}","${params.HubUser}")
                       }
                    }
                }
            }
        }


Git, maven, sonaqube and docker were integrated with Jenkins 

# Jenkins interface

<img src="images/Screenshot 2023-10-17 134028.png" alt="Alt text">

# Jenkins CICD stage

<img src="images/Screenshot 2023-10-17 133938.png" alt="Alt text">

# sonarqube interface after testing

<img src="images/Screenshot 2023-10-17 134353.png" alt="Alt text">

# poll SCM to smell code change

<img src="images/Screenshot 2023-10-17 134510.png" alt="Alt text">

# Pet-Clinic application

<img src="images/Screenshot 2023-10-17 134238.png" alt="Alt text">


# docker file 
        FROM openjdk:17
        VOLUME /app
        EXPOSE 8080
        ARG JAR_FILE=target/*.jar
        ADD ${JAR_FILE} app.jar
        ENTRYPOINT ["java","-jar","/app.jar"]
