# jenkins-docker-compose

Jenkins Docker Compose Setup Guide

* How I setup my jenkins 
    1 .SSL (HTTPS setup):
      Since I’ve mapped port 8443 for HTTPS in your Docker Compose, Jenkins will need to be configured with SSL certificates.
      I will either need to provide a self-signed certificate or obtain a certificate from a certificate authority
      (in my case  I will set up nginx proxy server and create certificatioon ). 
      After setting it up, make sure Jenkins is properly configured to point to these certificates.
    2.SSH:
      SSH will be used for secure connections between Jenkins master and agents. You already have SSH keys (strider_jenkins_key), 
      and you should ensure they are properly set up in the Jenkins configuration for SSH-based communication.
      Jenkins agents can authenticate using SSH keys to the master for secure and passwordless connections. You’ll need to set the correct SSH key in Jenkins when setting up the agent node.
    3.Master-Agent Mode:
      You’ve configured both the master and agent in Docker Compose. Since they’re running on the same network (jenkins-network), 
      communication between the two should be seamless, using the container names instead of IP addresses.
      You'll configure the master to communicate with the agent over SSH (which you've already prepared by setting environment variables like JENKINS_AGENT_SSH_PUBKEY).
    4.Using Docker Compose:
      Docker Compose allows easy management and deployment of both the master and agent. Since you have separate Dockerfiles for each component (master and agent), the configuration in Docker Compose is ready to scale.
      Compose will help in defining the volumes, networks, and dependencies needed for Jenkins to function across your setup, ensuring smooth operations and deployment.


























1) SSH Keys

    * SSH Key for Communication Between Jenkins Master and Agent:
        
        Step1)Generate an SSH key pair  (reference : https://www.jenkins.io/doc/book/using/using-agents/)
       
        Name the private key jenkins_agent_key.
        Register the private key in Jenkins Master as a credential.
        Set the public key as an environment variable in your Docker Compose file.
        During agent container creation, this public key will be added to the known_hosts file.
        Note: Ensure the private key has an extra carriage return at the end when uploaded to GitHub.

       

        Step2) New Node Setting 
        ![new node setting ](images/node-setting.png)
        * root directory : /home/jenkins/agent
        * Launch method : Launch agents via SSH
        * Host : localhost or ipaddress
        * HostkeyVerification Stratagy 
             option 1) Nonn verifying verification Stratagy  => nothing to do but not secure!!!
             option 2) Know host verification Stratagy => generate known_hosts file using ssh-keyscan
                ![How to add knownHost file to jenkins-master](images/ssh-keyscan.png)
                 change file permission to jenkins after create file
             option 3) MAnually trusted key Verification Stratagy 

    * SSH Key for GitHub:

        Use the id_ed25519 private key for communication with GitHub.
        Make sure this key is available on both Jenkins Agent and Master.
        Be cautious about any missing carriage returns in the private key content to prevent authorization errors.

2) Jenkins Master SSL Setup(Not neccessary when using a nginx proxy)

       Place the keystore.jks file inside the .ssh directory.
       To enable HTTPS on Jenkins Master, set the following environment variable in your Docker Compose file:

       ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore="/var/jenkins_home/.ssl/keystore.jks" --httpsKeyStorePassword="5785ch00"
  
       This will allow access through port 8443 while disabling HTTP for security reasons.


3) Docker in Docker Issue on Jenkins Agent

       Jenkins Agent may encounter permission issues when accessing /var/run/docker.sock.
       Although the Dockerfile for Jenkins Agent adds the Jenkins user to the Docker group within the agent container, 
       it doesn't affect the Docker group on the host machine.
          
       To resolve this, you can follow these steps:
       (Fith solution in https://phoenixnap.com/kb/docker-permission-denied) 
       Edit the Docker service file by running: sudo nano /usr/lib/systemd/system/docker.service
       Append the following lines to the bottom of the Service section:
     
       SupplementaryGroups=docker
       ExecStartPost=/bin/chmod 666 /var/run/docker.sock

       Restart the Docker service: sudo service docker restart
       Note: Changing permissions to 666 for /var/run/docker.sock is not recommended for security reasons, so the provided solution is a safer alternative.

       By following these instructions, you should be able to set up Jenkins with Docker Compose and resolve common SSH and Docker-related issues.


