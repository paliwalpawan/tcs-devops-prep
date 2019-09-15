<h4>Following steps describe the end-to-end process for completing TCS DevOps Golden Bridge Hackathon (2019) challenge</h4>
<h6>Note : For simplicity we used only three GCP VM instances</h6>

<h5>1) Software Provisioning Stage</h5>

```
VM instance creation 
	- Created a CentOs 7 instance <ANSIBLE> (with disk size as 30 GB)(allowed Allow full access to all Cloud APIs , allowed HTTP & HTTPS traffics under Firewall)
	- Created exactly similar instance for all other tools named <OTHERS> & <OTHERS2>


Creation of ssh keys in <ANSIBLE> instance 
	- Once <ANSIBLE> instance is created, clicked on Connect --> view gcloud command --> Copy the gcloud command --> RUN IN CLOUD SHELL --> Paste the copied gcloud command --> Hit enter 
	- sudo passwd root (change root password to "root")
	- su - root
	- ssh-keygen (hit enter when prompted)
	- copy id_rsa.pub content --> go to VM instances --> click on <OTHERS> VM instance  --> Edit --> Go to SSH Keys section --> paste the content of id_rsa.pub there --> save --> repeat the same process for <OTHERS2> instance as well

Allowing root login in <OTHERS> & <OTHERS2> instance from <ANSIBLE> instance
	- On <OTHERS> instance , clicked on Connect --> view gcloud command --> Copy the gcloud command --> RUN IN CLOUD SHELL --> Paste the copied gcloud command --> Hit enter 
	- sudo passwd root (change root password to "root")
	- su - root
	- vim /etc/ssh/sshd_config --> change PermitRootLogin to 'yes'
	- service sshd restart
	- repeat the same steps for <OTHERS2> instance as well

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
|	jenkins						ansible-galaxy install geerlingguy.jenkins (https://galaxy.ansible.com/geerlingguy/jenkins)
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

Installation of Java, Maven , Jenkins , Docker in <OTHERS> using ansible
	- In <ANSIBLE> instance, switch to root (su - root)
	- Execute following command :
		ansible-galaxy install geerlingguy.java gantsign.maven geerlingguy.jenkins geerlingguy.docker
	- Then copy the contents of ansible-playbooks/install-java-maven-jenkins-docker.yaml in <ANSIBLE> VM
	- Execute following command
		ansible-playbook install-java-maven-jenkins-docker.yaml
	- Ensure there are no error messages
	- Login to <OTHERS> VM
	- Check whether installations are successful
		Java : java -version , javac -version
		Maven : mvn --version
		Docker: docker --version
		Jenkins: service jenkins status 
	- Now, access Jenkins from browser : "http://<EXTERNAL IP OF OTHERS VM>:9000"

Installation of Java, Sonarqube, jFrog artifactory in <OTHERS2> using ansible
	- In <ANSIBLE> instance, switch to root (su - root)
	- Then copy the contents of ansible-playbooks/install-java-jfrog-sonarqube.yaml in <ANSIBLE> VM
	- Execute following command
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
