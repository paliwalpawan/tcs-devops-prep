<h4>Following steps describe the end-to-end process for completing TCS DevOps Golden Bridge Hackathon (2019) challenge</h4>
<h6>Note : For simplicity we used only three GCP VM instances</h6>


<h4>Index</h4>
- <a href="#software-provisioning-stage">1) Software Provisioning Stage</a><br/>
- <a href="#jenkins-pipeline">2) Jenkins Pipeline</a><br/>
- <a href="#required-configurations">3) Required Configurations</a></br>

<h5 id="software-provisioning-stage">1) Software Provisioning Stage</h5>

```
VM instance creation 
	- Created a CentOs 7 instance <ANSIBLE> (with disk size as 30 GB)(allowed Allow full access to all Cloud APIs , allowed HTTP & HTTPS traffics under Firewall)
	- Created exactly similar instance for all other tools named <OTHERS> & <OTHERS2>
	
Kubernetes cluster creation 
	- Created a kubernetes cluster with 3 nodes


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
	- Now, in <ANSIBLE> instance command prompt, try "ssh root@public or private IP of OTHERS>" and verify that passwordless ssh is indeed happening
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
|	java 8 (for nexus) java 11+ (for sonarqube) ansible-galaxy install geerlingguy.java (https://galaxy.ansible.com/geerlingguy/java)
|	sonarqube					
|	sonatype nexus					ansible-galaxy install bbaassssiiee.nexus (https://galaxy.ansible.com/bbaassssiiee/nexus)			
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
		cd  <repository directory>/ansible-playbooks
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

Installation of Java, Sonarqube, Nexus artifactory in <OTHERS2> using ansible
	- In <ANSIBLE> instance, switch to root (su - root)
	- Execute following commands in <ANSIBLE> VM:
		git clone <repository url>
		cd  <repository directory>/ansible-playbooks
		Verify the VM where nexus will be installed is specified in deamon.json - IMPORTANT !!!
		ansible-playbook install-java-jfrog-sonarqube.yaml
	- Ensure there are no error messages
	- Login to <OTHERS2> VM
	- Check whether installations are successful
		Java : java -version , javac -version
		Sonar: ls /opt/sonar*
		nexus: sudo service nexus status 
	- I was unable to start Sonarqube as service, so it had to be mannually started with following steps :
		su - sonarqube (password : sonarqube)
		cd /opt/sonar*/bin/linux*/
		./sonar.sh start
	- Now, access Sonarqube from browser : "http://<EXTERNAL IP OF OTHERS2 VM>:9000" (default credentials admin / admin)
	- And, access nexus from browser : "http://<EXTERNAL IP OF OTHERS2 VM>:8081" (Look out for initial password in the login screen & then set new admin password credentials admin / admin)
```

<h5 id="jenkins-pipeline">2) Jenkins Pipeline</h5>

![Jenkins Pipeline](https://github.com/ArghyaChakraborty/tcs-devops-hackathon-prep-project/raw/master/images/Jenkins-Pipeline.JPG)

<h5 id="required-configurations">3) Required Configurations </h5>

```
		
SONARQUBE:
	Open sonarqube in browser with "http://<EXTERNAL IP OF OTHERS2 VM>:9000"
	Create new project --> enter some "Project key" --> Set up --> Enter some text in "Generate Token" --> Take a note of the token (IMPORTANT !!) --> Continue --> Select Project's main language as "Java" --> Select build technology as "maven" --> take a note of the scanner command (IMPORTANT !)
	
NEXUS:
	Open nexus in browser with "http://<EXTERNAL IP OF OTHERS2 VM>:8081"
	Click on Settings (cog-wheel icon in the top menu bar) --> Reositories --> either create a "hosted" maven rpository OR use an existing one (ex. maven-releases)
	Click on Settings (cog-wheel icon in the top menu bar) --> Reositories --> Create repository --> docker (hosted) --> provide a name --> ensure the "Online" checkbox is checked --> Under "Repository Connectors" , check "HTTP" -> provide port# 8083 (DO NOT CHANGE IT SINCE THE SMAE PORT IS SPECIFIED IN ansible-playbooks/daemon.json) --> keep remaining settings as-is --> Save
	Click on Settings (cog-wheel icon in the top menu bar) --> Security --> Realms -> Add "Docker bearer token realm" to the Active list (right hand side panel) 
	
JENKINS : 
	Open Jenkins with "http://<EXTERNAL IP OF OTHERS VM>:8080"
	Provide initialAdminPassword
	Install suggested plugins
	Create admin user (admin / admin)
	After jenkins start, got to Manage Jenkins --> Global Tool Configuration
		- Add configurations for JDK, Maven , Git & Docker
	Go to Manage Jenkins --> Manage Plugins --> Available tab --> Install following (Install without restart)
		- Blue Ocean (for nice visualization of pipeline stages)
		- Nexus Platform (for uploading / downloading files to/from Nexus artifactory)
		- Google kubernetes engine 
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
	Go to Manage Jenkins --> Configure system --> Go to "Sonatype Nexus" section --> Add nexus repository manager server --> Provide following details :
		Display name  :
		Server ID : 
		Server URL : "http://<EXTERNAL IP OF OTHERS2 VM>:8081"
		Credentials : Add a new jenkins credential to reflect nexus admin credentials
	--> Save
	PERFORM THE STEPS WRITTEN IN THIS URL : https://github.com/jenkinsci/google-kubernetes-engine-plugin/blob/master/docs/Home.md (IMPORTANT !!!!). Follow ONLY below mentioned sections from this page :
		- Enable Required APIs
		- IAM Credentials
		- Configure GKE Cluster Step 2 (Step 1 will be performed from GKE UI)
		- Configure Kubernetes Cluster Permissions (Follow Less Restrictive Permissions section. Skip More Restrictive Permissions , Automation Option & References sections),
		- Ignore rest of the steps of the page
	Before creating the Jenkins Pipeline
		- Visit the source code repository & update URLs, project name , credentials etc in all the files , especially Jenkinsfile
	Go to Jenkins dashboard --> New Item -> Enter name of the pipeline Job --> Select Pipeline --> Ok --> In the [Pipeline] section, select following values :
		- Definition: pipeline script from SCM
		- SCM: Git
		- Add repository details
		- Save
```
