FROM jenkins/jenkins:latest
USER root  

COPY ["./.ssl/" ,"/var/jenkins_home/.ssl"]
RUN chown -R jenkins:jenkins /var/jenkins_home/.ssl
COPY ["./.ssh/" ,"/var/jenkins_home/.ssh"]
RUN chown -R jenkins:jenkins /var/jenkins_home/.ssh && chmod 700 /var/jenkins_home/.ssh/ && chmod 600 /var/jenkins_home/.ssh/*

# Add GitHub's host key to known_hosts
RUN ssh-keyscan github.com >> /var/jenkins_home/.ssh/known_hosts

ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore="/var/jenkins_home/.ssl/keystore.jks" --httpsKeyStorePassword="5785ch00"

USER jenkins

# Use the ssh-keyscan script to create known_host file 
#ENTRYPOINT ["/var/jenkins_home/.ssh/ssh-keyscan.sh"]