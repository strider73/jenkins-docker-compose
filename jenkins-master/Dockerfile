FROM jenkins/jenkins:latest
USER root  

COPY ["./.ssl/" ,"/var/jenkins_home/.ssl"]
RUN chown -R jenkins:jenkins "/var/jenkins_home/.ssl"
COPY ["./.ssh/" ,"/var/jenkins_home/.ssh"]
RUN chown -R jenkins:jenkins "/var/jenkins_home/.ssh"



ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore="/var/jenkins_home/.ssl/keystore.jks" --httpsKeyStorePassword="5785ch00"
# ENV JENKINS_OPTS  --httpsPort=8443 --httpsKeyStore="/var/jenkins_home/.ssl/keystore.jks" --httpsKeyStorePassword="5785ch00"

USER jenkins
