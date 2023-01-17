
Simple DevOps End to End Project 

1. What is Jenkins? Jenkins is a glue in the project. It's used for autometed builds and CI/CD. 
2. Spin up EC2 and configure as a Jenkins Server 
    2.1 sudo su - 
    2.2 vim /etc/hostname --> dd --> i --> jenkins-server --> wq --> reboot 
    2.3 wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    2.4 rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    2.5 yum upgrade
    2.6 amazon-linux-extras install java-openjdk11 -y
    2.7 amazon-linux-extras install epel  (Extra packages enterprise linux)
    2.8 systemctl enable jenkins
    2.9 systemctl start jenkins
    3.0 copy public ip of jenkins vm and <public-ip>:8080
    3.1 Make sure 8080 is open in SG level 
    3.2 cat /var/lib/jenkins/secrets/initialAdminPassword


Scenario: As a DevOps engineer, we work closely with Developers. Developers develop Java based apps. 
As a DevOps engineer we need to take Java code from GitHub repo, build war/jar build artifacts and deploy 
to Target Serves (EC2). 
    Steps for solution:

    Configure Maven 
    1. Configure Maven in Jenkins. Maven is a build management tool 
    2. cd /opt 
    3. wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
    4. tar -xzvf apache-maven-3.8.6-bin.tar.gz 
    5. rm -Rf apache-maven-3.8.6-bin.tar.gz 
    6. mv apache-maven-3.8.6/ maven
    7. cd ~ --> go to root 
    8. vim .bash_profile 
        User specific environment and startup programs
        JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.16.0.8-1.amzn2.0.1.x86_64
        M2_HOME=/opt/maven
        M2=$M2_HOME/bin
        PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
    By configuring .bash_profile and exporting M2_HOME, M2 and JAVA_HOME --> commands can be executed from everywhere. 
    9. source .bash_profile
    10. Jenkins GUI --> Manage Jenkins --> Manage Plugins --> Maven Integration --> Install without restart 
    11. clean install --> each time job runs it cleans the cache/memory 
    12. in Jenkins VM jobs are stored under /var/lib/jenkins/workspace

    Tomcat 
    1. What is Tomcat?  Apache Tomcat is a web container. It allows the users to run Servlet and 
    JAVA Server Pages that are based on the web-applications
    2. cd /opt 
    3. wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.68/bin/apache-tomcat-9.0.68.tar.gz
    4. tar -xzvf apache-tomcat-9.0.68.tar.gz
    5. rm -Rf apache-tomcat-9.0.68.tar.gz
    6. mv apache-tomcat-9.0.68 tomcat 
    7. cd tomcat/bin and you will see lots of files 
    8. ./startup.sh is for starting up a tomcat server
    9. publicIP:8080
    10. Click on Manager App —> Access Denied 
    11. find / -name context.xml
        <!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|i0:0:0:0:0:0:0:1" /> -->
    12. cd conf —> vi tomcat-users.xml —> to the end
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>
    <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
    <user username="deployer" password="deployer" roles="manager-script"/>
    <user username="tomcat" password="s3cret" roles="manager-gui"/>
    </tomcat-users>
    13. cd ../bin —> ./shutdown.sh —> ./startup.sh 
    


KISS (Keep it Simple and Stupid )
    11/10/2022 Thursday 

    1. Difference between VM (Virtual Machine) and Container? VMs are heavywait but Containers are lightweight
    2. Lightweight means easy to deploy, migrate and test 
    3. Each VM use their own OS, Containers can use only single OS 
    4. Microservices --> Loosely couple apps vs tightly coupled apps 
    5. Microservices are Loosely coupled apps --> easy to detect the bugs, easy to fix. 
    6. In Microservices --> AppA, AppB and AppC --> If AppB fails AppA and AppC won't fail. Its totally 
    opposite in legasy apps (tightly coupled apps)
    7. tightly coupled apps called monolithic app.  

    Greatest candidate for Microservices is container. 
    1. Container runtimes -->  Docker, Rocket. Docker is the most famous and we will only learn Docker. 

Configure docker 

1. EC2 vm spin up 
2. sudo su - 
3. yum install docker -y 
4. docker images --> Will fail 
5. service docker status --> returns inactive (dead)
6. service docker start 
7. create docker hub account --> https://hub.docker.com/
8. search box look for tomcat image 
9. docker pull tomcat
10. Docker container can run in 2 modes --> 1) Foreground and 2) detached 
11. Foreground --> docker run --name tomcat-container -p 8080:8080 tomcat:latest --> If proccess is canceled docker container will die 
12. Detach --> docker run -d --name tomcat-container-1 -p 8081:8080 tomcat:latest
13. List docker containers --> docker ps 
14. List all container icluding exited containers --> docker ps -a 
15. docker container ls or docker container ls -a
16. Remove container --> docker stop container-name or id --> docker rm container-name or id
17. Remove image --> docker rmi image name or image id 
18. Dockerfile --> Is a definition for docker image 


11/15/2022          Tuesday 

Jenkins/Docker/Tomcat cont. 

1. SSH into docker vm and create /opt/docker dir 
2. create dockeradmin user --> useradd dockeradmin and passwd dockeradmin
3. cat /etc/passwd 
4. cat /etc/group 
5. usermod -aG docker dockeradmin --> Adds dockeradmin to docker group 
6. In Jenkins server --> Install Publish_over_ssh plugin
7. Configure System --> SSH server and pass all required fields. 
8. In Docker VM --> vim /etc/ssh/sshd_config 
    # To disable tunneled clear text passwords, change to no here!
    PasswordAuthentication yes
    #PermitEmptyPasswords no
    #PasswordAuthentication no
9. service sshd reload 
10. Jenkins GUI --> new item --> Name: deploy_on_container --> Source files: **/*.war
Remove prefix: /webapp/target --> Remote directory: //opt//docker 
11. Docker VM --> sudo su - --> chown -R dockeradmin:dockeradmin /opt/docker 
12. Docker VM --> Dockerfile 
13. docker build -t register-image .
14. docker run -d --name register-container -p 8080:8080 register-image
15. Jenkins GUI --> Exec command --> cd /opt/docker;
                    docker build -t register-image .;
                    docker stop register-container;
                    docker rm register-container;
                    docker run -d --name register-container -p 8080:8080 register-image;


Ansible 
1. Configuration Management/Deployment tool and it uses yml/yaml format 
2. Built on top of Python 
3. It's agentless --> all you need is ssh keys 
4. Make sure Python is installed 
5. pip3 install ansible 
6. useradd ansadmin 
7. passwd ansadmin
8. install docker in the vm 
9. usermod -aG docker ansadmin --> adds ansadmin user to docker group 
10. id ansadmin
11. visudo 
    ## Allow root to run any commands anywhere 
    root    ALL=(ALL)       ALL
    ansadmin ALL=(ALL)       NOPASSWD:ALL
12. vim /etc/ssh/sshd_config
        # To disable tunneled clear text passwords, change to no here!
        PasswordAuthentication yes
        #PermitEmptyPasswords no
        #PasswordAuthentication no
13. yum install docker -y 
14. ssh-keygen 
15. su - ansadmin
16. ssh-copy-id <private ip of docker vm>
17. ansible all -m ping --> test connection
18. mkdir /etc/ansible 
19. vim hosts --> add private ip of docker vm 


11/17/2022 Thursday

1. ansible all -m ping (Ad hoc command) --> Checks the connectivity from Ansible controller to Ansible 
managed VMs 
2. ssh-copy-id <private ip>

Scenario: How do you share docker images across your team? Dockerhub