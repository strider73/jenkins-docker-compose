# jenkins-docker-compose

Jenkins Docker Compose Setup Guide

1) SSH Keys

    * SSH Key for Communication Between Jenkins Master and Agent:
        
        Step1)Generate an SSH key pair  (reference : https://www.jenkins.io/doc/book/using/using-agents/)
       
        Name the private key jenkins_agent_key.
        Register the private key in Jenkins Master as a credential.
        Set the public key as an environment variable in your Docker Compose file.
        During agent container creation, this public key will be added to the known_hosts file.
        Note: Ensure the private key has an extra carriage return at the end when uploaded to GitHub.

        ![How to add knownHost file to jenkins-master](images/ssh-keyscan.png)

        Step2) Node Setting 
        ![new node setting ](images/node-setting.png)

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


