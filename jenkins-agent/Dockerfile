# This is jenkins ssh agent
# 1. Docker command available
# 2. Docker Compose installed
# 3. GitHub SSH credential is ready
FROM jenkins/ssh-agent:jdk17

# Install prerequisites
USER root
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Install Docker for x86_64 (64-bit)
RUN curl -fsSL https://get.docker.com -o get-docker.sh
RUN sh get-docker.sh

# Install Docker Compose
RUN curl -fsSL https://github.com/docker/compose/releases/download/v2.20.3/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

# Install jq
RUN apt-get update && apt-get install -y jq
RUN apt-get update && apt-get install -y openssh-client


# Add Jenkins user to Docker group
RUN usermod -aG docker jenkins

# Install nano (optional)
RUN apt update
RUN apt install nano
# Install Maven
RUN apt-get update && apt-get install -y maven

# Copy SSH private key for git hub 
#COPY .ssh/id_ed25519 /home/jenkins/.ssh/id_ed25519
COPY ["./.ssh/" ,"/home/jenkins/.ssh"]
RUN chown -R jenkins:jenkins /home/jenkins/.ssh && chmod 600 /home/jenkins/.ssh/id_ed25519 && chmod 644 /home/jenkins/.ssh/id_ed25519.pub 

# Add GitHub's host key to known_hosts
RUN ssh-keyscan github.com >> ~/.ssh/known_hosts


# Switch back to Jenkins user
USER jenkins
