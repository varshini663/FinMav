pipeline{
agent any
tools{
maven 'Maven'
}
stages{
stage('Checkout'){
steps{
git branch:'master',url:'https://github.com/varshini663/FinMav.git'
}
}
stage('Build'){
steps{
sh 'mvn clean package'
}
}
stage('Test'){
steps{
sh 'mvn test'
}
}
stage('Run'){
steps{
sh 'java -jar target/FinMav-1.0-SNAPSHOT.jar'
}
}
}
post{
success{
echo 'Successful'
}
failure{
echo 'not built'
}
}
}

To create a Jenkins pipeline for maven based webapp using ansible to automatically copy the war file from Maven target folder to tomcat webapps folder and run the web application

1. Make sure apache tomcat is installed in the root directory

2. Add the tomcat.service file if not available
2.1 Set Permissions for the username hemavathi
// id tomcat

sudo chmod +x /opt/tomcat/bin/*.sh
//sudo chown -R tomcat:hemavathi /opt/tomcat
sudo chmod +x /opt/tomcat/bin/*.sh
//sudo chmod +x /opt/tomcat/bin/*.sh

2.2. write the tomcat.service file
sudo -i   // go to root directory
cd /
cd etc/systemd/system
ls
//please check for the tomcat.service file,if available ignore the below steps,if not do add the tomcat.service file 

2.3. gedit tomcat.service

After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target


2.4 Now we can start and stop the tomcat server directly from user instead of from root directory
hemavathi@hemavathi-VirtualBox:~$ sudo systemctl stop tomcat
hemavathi@hemavathi-VirtualBox:~$ sudo systemctl start tomcat


3. while executing, Sometimes it says permission denied for creating pipeline for webapplication on jenkins so kindly follow the commands 
//Downloads the latest official Jenkins GPG key and Saves the key in /usr/share/keyrings/ so that apt can use it for verification.

hemavathi@hemavathi-VirtualBox:~$ curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

// Associate the Repository with the New Key
After adding the key, we must explicitly tell the system to use it for verifying Jenkins packages: 
This ensures that only trusted Jenkins packages will be installed.

hemavathi@hemavathi-VirtualBox:~$ echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

hemavathi@hemavathi-VirtualBox:~$ sudo apt update

after this restart the jenkins
localhost:8080/safeRestart


4. Create a maven based web application

4.1 mvn archetype:generate -DgroupId=com.example -DartifactId=MavenAnsibleWebApp -DarchetypeArtifactId=maven-archetype-webapp -DinterativeMode=false

4.2 cd MavenAnsibleWebApp
gedit pom.xml  // add servet dependency from maven repository and change the plugin to war type 

This XML file does not appear to have any style information associated with it. The document tree is shown below.
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.example</groupId>
<artifactId>MavenAnsibleWebApp</artifactId>
<packaging>war</packaging>
<version>1.0-SNAPSHOT</version>
<name>MavenAnsibleWebApp</name>
<url>http://maven.apache.org</url>
<dependencies>
<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>3.8.1</version>
<scope>test</scope>
</dependency>
<!--  Servlet API for Java EE  -->
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>4.0.1</version>
<scope>provided</scope>
</dependency>
</dependencies>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-war-plugin</artifactId>
<version>3.4.0</version>
<!--  Update to latest stable version  -->
</plugin>
</plugins>
<finalName>MavenAnsibleWebApp</finalName>
<!--  create a web page with the name MavenAnsibleWebApp, if we want some other name we can chage it  -->
</build>
</project>


4.3 mvn clean install  //generates a war file in target folder MavenAnsibleWebApp.war
4.4 create a new git repositoy MavenAnsibleWebApp
4.5 in terminal type gedit Jenkinsfile

pipeline {
    agent any  // Use any available agent
    
    environment {
        LANG = 'en_US.UTF-8'
        LC_ALL = 'en_US.UTF-8'
    }   // this has to be added only if you get an error saying UTF required is 8 but showing in ISO00009

    tools {
        maven 'Maven'  // Ensure this matches the name configured in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Hemavathipcse/MavenAnsibleWebApp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'  // Run Maven build
            }
        }

     stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint:true
            }
        }
        stage('Deploy') {
            steps {
               sh 'mvn clean package'  
               sh 'ansible-playbook ansible/playbook.yml -i ansible/hosts.ini'
            }
        }

                  
    }

   }
   

4.6 Create a Ansible host file and playbook/deploy file 
we can either create a ansible directory within the maven project folder and then add yaml and ini files or directly within the maven project folder ( accordingly we have to make the changes in the jenkins file ie sh ' ansible-playbook ansible/playbook.yml -i ansible/hosts.ini' or sh ansible-playbook playbook.yml -i hosts.ini')

mkdir ansible
cd ansible
hemavathi@hemavathi-VirtualBox:~/MavenAnsibleWebApp/ansible$ gedit hosts.ini

[web]
localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python3

save and exit

hemavathi@hemavathi-VirtualBox:~/MavenAnsibleWebApp/ansible$ gedit playbook.yml


---
- name: Deploy to Tomcat
  hosts: web
  become: yes
  tasks:
    - name: Copy WAR file to Tomcat
      copy:
        src: "/var/snap/jenkins/4829/workspace/MavenAnsible-CICD/target/MavenAnsibleWebApp.war"
        dest: "/opt/tomcat/webapps/MavenAnsibleWebApp.war"
        remote_src: no

    - name: Restart Tomcat
      systemd:
        name: tomcat
        state: restarted
        
save and exit
//Note: in the above script we need to check two things 
    a) the number 4829, please write this number with to respect to your sytem generated folder number  
    b)MavenAsnible-CICD is the pipeline name which is created in the jenkins ( create new item and give the name as your wish --the respective name has to be specified here)  
    
    
5. Push it on to git hub

git init
git add.
git commit -m "Initial commit"
git remote add origin https://github.com/Hemavathipcse/MavenAnsibleWebApp.git
git remote set-url origin @github.com/Hemavathipcse/MavenAnsibleWebApp.git
git push -u origin master


//please checkout in the git hub repository

6. create a jenkins pipeline

6.1 goto browser and type localhost:8080   // logs into jenkins
6.1 click on new item and give the name MavenAnsible-CICD // it should match with the name given in the playbook.yml
    select pipeline from the item item and click ok
    
6.2  we get one more window which will ask us to configue the pipeline 
   6.2.1 Description //give the description as you require
   6.2.2 scroll down, we can pipeline section in this 
          Definition: pipeline script from SCM(source Code Manager)  //select this option
          SCM: Git  //select
          Repository URL : https://github.com/Hemavathipcse/MavenAnsibleWebApp.git //copy and paste the git hub repository url
          Credentials: MAvenGradleCICD // displays the name which you have given while creating the token 
          Click on lighweight checkout // checkbox
          Apply and ok
          
6.3 Displays the pipeline which we created just now 
     select and click on build now, 
     //displays build is scheduled and shows the pipeline stages what we have written in jenkinsfile 
     //either successful or build failed. If failed we can click on logs and see the errors and fix the bugs accordingly and rebuild
     //if successful, go to logs and click on the last line it shows the respective output
     
     
 

     
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   
   
   
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   
   





