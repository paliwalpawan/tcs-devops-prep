<h4>Following steps describe the end-to-end process for completing TCS DevOps Golden Bridge Hackathon (2019) challenge</h4>
<h6>Note : For simplicity we used only three GCP VM instances</h6>

<h5>1) Software Provisioning Stage</h5>

```
VM instance creation 
	- Created a CentOs 7 instance <ANSIBLE> (with disk size as 30 GB)(allowed Allow full access to all Cloud APIs , allowed HTTP & HTTPS traffics under Firewall)
	- Created exactly similar instance for all other tools named <OTHERS> & <OTHERS2>
	
Kubernetes cluster creation 
	- Created a kubernetes cluster with 1 nodes & 1 CPU


Creation of ssh keys in <ANSIBLE> instance 
	- Once <ANSIBLE> instance is created, clicked on Connect --> view gcloud command --> Copy the gcloud command --> RUN IN CLOUD SHELL --> Paste the copied gcloud command --> Hit enter 
	- sudo passwd root (change root password to "root")
	- su - root
	- ssh-keygen (hit enter when prompted)
	- copy id_rsa.pub content --> go to VM instances --> click on <OTHERS> VM instance  --> Edit --> Go to SSH Keys section --> paste the content of id_rsa.pub there --> save --> repeat the same process for <OTHERS2> instance as well --> repeat the same process for <ANSIBLE> (so that the server can ssh into itself)

Allowing root login in <ANSIBLE>, <OTHERS> & <OTHERS2> instance from <ANSIBLE> instance
	- On <OTHERS> instance , clicked on Connect --> view gcloud command --> Copy the gcloud command --> RUN IN CLOUD SHELL --> Paste the copied gcloud command --> Hit enter 
	- sudo passwd root (change root password to "root")
	- su - root
	- vim /etc/ssh/sshd_config --> change PermitRootLogin to 'yes'
	- service sshd restart
	- repeat the same steps for <ANSIBLE> & <OTHERS2> instance as well

Verification of passwordless ssh from <ANSIBLE> -> <OTHERS>
	- Now, in <ANSIBLE> instance command prompt, try "ssh root@<public or private IP of OTHERS>" and verify that passwordless ssh is indeed happening
	- Thus <ANSIBLE> -> <OTHERS> passwordless ssh is established
	- Since the configuratio is same, <ANSIBLE> -> <OTHERS2> should also work
	- The exact same process can be followed if we need to enable <OTHERS> -> <ANSIBLE> passwordless ssh
	
Modifying default firewall rules
	- Go to Networking --> VPC network --> Firewall rules
	- edit rule "default-allow-http" & change "Protocols and ports" to "all" (default : tcp:80)
	- edit rule "default-allow-https" & change "Protocols and ports" to "all" (default : tcp:443)
	- Login to <ANSIBLE> instance 
	- run "python -m SimpleHTTPServer 8000"
	- Now, try to access "http://<EXTERNAL IP OF ANSIBLE VM>:8000"
	- If it's accessible, then everything is good to go
	
Sofwares that need installation (as per my understanding) :
 ------ <use ANSIBLE VM for this>--------------------------------------
|	ANSIBLE (need manual installation)
 ----------------------------------------------------------------------
 ------ <use OTHERS VM for all these>----------------------------------
|	JDK 8 (for jenkins & maven)		ansible-galaxy install geerlingguy.java (https://galaxy.ansible.com/geerlingguy/java)
|	maven (for jenkins)			ansible-galaxy install gantsign.maven (https://galaxy.ansible.com/gantsign/maven)
|   git (for jenkins)
|	jenkins						
|	docker						ansible-galaxy install geerlingguy.docker (https://galaxy.ansible.com/geerlingguy/docker)
 ----------------------------------------------------------------------
 ------<use OTHERS2 VM for all these>----------------------------------
|	java 11 (for sonarqube & jFrog) ansible-galaxy install geerlingguy.java (https://galaxy.ansible.com/geerlingguy/java)
|	sonarqube					
|	jFrog artifactory			
 ----------------------------------------------------------------------

Installation of ansible in <ANSIBLE> instance
	- To install ansible , do following steps in console (as root user) :
		sudo yum install epel-release
		sudo yum install ansible
	- Edit hosts file to add IP address of <OTHERS> Vm instance
		vim /etc/ansible/hosts
		anywhere in the file add following section 
			[others]
			<private IP of OTHERS VM>
			[others2]
			<private IP of OTHERS2 VM>
	- Test ansible installation
		ansible others -m ping
    - Also, install gitlab client 
		yum install git

Installation of Java, Maven , Git, Jenkins , Docker in <OTHERS> using ansible
	- In <ANSIBLE> instance, switch to root (su - root)
	- Execute following commands in <ANSIBLE> VM:
		git clone <repository url>
		cd  <repository directory>
		ansible-playbook install-java-maven-git-jenkins-docker.yaml
	- Ensure there are no error messages
	- Login to <OTHERS> VM
	- Check whether installations are successful
		Java : java -version , javac -version
		Maven : mvn --version
		Docker: docker --version
		Git: git --version
		Jenkins: service jenkins status 
	- Now, access Jenkins from browser : "http://<EXTERNAL IP OF OTHERS VM>:8080"

Installation of Java, Sonarqube, jFrog artifactory in <OTHERS2> using ansible
	- In <ANSIBLE> instance, switch to root (su - root)
	- Execute following commands in <ANSIBLE> VM:
		git clone <repository url>
		cd  <repository directory>
		ansible-playbook install-java-jfrog-sonarqube.yaml
	- Ensure there are no error messages
	- Login to <OTHERS2> VM
	- Check whether installations are successful
		Java : java -version , javac -version
		Sonar: ls /opt/sonar*
		jFrog: sudo service artifactory status
	- I was unable to start Sonarqube as service, so it had to be mannually started with following steps :
		su - sonarqube (password : sonarqube)
		cd /opt/sonar*/bin/linux*/
		./sonar.sh start
	- Now, access Sonarqube from browser : "http://<EXTERNAL IP OF OTHERS2 VM>:9000" (default credentials admin / admin)
	- And, access jFrog from browser : "http://<EXTERNAL IP OF OTHERS2 VM>:8081" (default credentials admin / password)
```

<h5>2) Jenkins Pipeline</h5>

![Jenkins Pipeline](https://github.com/ArghyaChakraborty/tcs-devops-hackathon-prep-project/raw/master/images/Jenkins-Pipeline.JPG)

<h5>3) Preparation work </h5>

```
		
SONARQUBE:
	Open sonarqube in browser with "http://<EXTERNAL IP OF OTHERS2 VM>:9000"
	Create new project --> enter some "Project key" --> Set up --> Enter some text in "Generate Token" --> Take a note of the token (IMPORTANT !!) --> Continue --> Select Project's main language as "Java" --> Select build technology as "maven" --> take a note of the scanner command (IMPORTANT !)
	
jFROG:
	Open artifactory in browser with "http://<EXTERNAL IP OF OTHERS2 VM>:8081"
	At the top right corner, Click on the "Welcome, admin" dropdown --> Select Create Repositories --> Local Repository --> Select package type as "Generic" --> Provide some "Repository Key" --> Save & Finish
	
JENKINS : 
	Open Jenkins with "http://<EXTERNAL IP OF OTHERS VM>:8080"
	Provide initialAdminPassword
	Install suggested plugins
	Create admin user (admin / admin)
	After jenkins start, got to Manage Jenkins --> Global Tool Configuration
		- Add configurations for JDK, Maven , Git & Docker
	Go to Manage Jenkins --> Manage Plugins --> Available tab --> Install following (Install without restart)
		- Blue Ocean (for nice visualization of pipeline stages)
		- Artifactory (for uploading / downloading files to/from jFrog artifactory)
	Go to Manage Jenkins --> Manage Nodes --> New Node
		- Prerequisite : The slave node(s) must have password login enabled (IMPORTANT !!!)
			> Login to all the target slave nodes
			> su - root
			> vim /etc/ssh/sshd_config
			> Change "PasswordAuthentication" to yes (PasswordAuthentication yes)
			> service sshd restart
		- Node Name : slave-others2
		- Click on "Permanent Agent"
		- Ok
		- Remote root directory : /var/
		- Labels: slave
		- Host: <External IP of OTHERS2 VM>
		- Credentials --> Add --> Add root credentials for OTHERS2 VM
		- Host Key Verification Strategy : Non verifying....
		- Save
	Go to Manage Jenkins --> Configure system --> Go to "Artifactory" section --> Add artifactory server --> Provide following details :
		Server Id
		URL (url should be like http://34.70.150.87:8081/artifactory/)
		Default deployer username 
		Default deployer password
	--> Save

	
	
..........
```
