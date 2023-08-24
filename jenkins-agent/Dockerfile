
#This is jenkins ssh agent  
# 1. docker command available 
# 2. and docker compose installed 
# 3. github ssh credential is ready 
FROM jenkins/ssh-agent:jdk17

# Install prerequisites
USER root
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Install Docker for ARM64
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
RUN echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update && apt-get install -y docker-ce-cli

RUN curl -fsSL https://github.com/docker/compose/releases/download/v2.20.3/docker-compose-Linux-aarch64 -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose


# Install jq
RUN apt-get update && apt-get install -y jq

# Add Jenkins user to Docker group
RUN groupadd -g 995 docker
RUN usermod -aG docker jenkins

# Copy SSH private key
COPY .ssh/id_ed25519 /home/jenkins/.ssh/id_ed25519

# Switch back to Jenkins user
USER jenkins
