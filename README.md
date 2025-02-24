# AdventureTube CI/CD Pipeline Documentation

## Overview

AdventureTube utilizes a backend system comprised of microservices. Each microservice is independently built as a Docker container and managed using Docker Compose. While this architecture ensures modularity and streamlined updates, the build and test processes can be repetitive and time-consuming. To automate and optimize these processes, Jenkins has been integrated to establish a **Continuous Integration and Continuous Deployment (CI/CD) pipeline**, ensuring efficiency, consistency, and scalability in the development workflow.

The AdventureTube project follows a structured CI/CD pipeline to automate building, testing, and deploying microservices. The pipeline is designed to work with two Raspberry Pi devices:

- **Pi1**: Runs Jenkins Master, PostgreSQL, MongoDB, and Kafka.
- **Pi2**: Runs Jenkins Agent , microservices and Spring Cloud components (Eureka, Config Server, Gateway).

---

## **Jenkins Master/Agent Architecture**

Jenkins master/agent architecture is the best choice for testing conditions in a completely isolated environment. The master handles **only orchestration**, while the agent executes the tasks. This guarantees job performance and clear separation.

Most importantly, **it provides complete toolset configuration freedom**—you can configure and modify environments without affecting other builds. This flexibility is crucial for future automation in CI/CD pipelines.

---

## **Jenkins Master/Agent SSH Connection Setup**

SSH is used for secure connections between the Jenkins master and agents. Pre-made SSH keys (`jenkins_agent_key`) are set up for authentication within the Jenkins configuration.

Jenkins agents typically authenticate using SSH keys under the **"Launch agent by connecting it to the controller"** option. However, in this setup, **"Launch agents via SSH"** is used instead. This allows the **controller to initiate, control, and remove agents dynamically**.

### **Key Differences in SSH Setup**

- **Jenkins Master:** Holds the **private key**. => SSH Server
- **Jenkins Agent:** Holds the **public key**. => SSH Client 

This is the **opposite** of the standard Jenkins setup where the agent connects to the master.

During the configuration of **"Launch agents via SSH"**, you must understand the **Host Key Verification Strategy**. This setting applies to connections between the master and agent.(detail in below : Host Key Verification Strategy)


## **Step 1: SSL Configuration**

In Jenkins, when configuring SSL, you typically need to handle two main components:

### **SSL Certification Configuration**

- **Using an [Nginx proxy server](https://github.com/strider73/nginx) as a reverse proxy** (Personaly I do way more prefer ).
- **Self-signed certificates to the Java Keystore.**

### **Port Configuration**

To enable HTTPS on Jenkins Master, set the following environment variable in your Docker Compose file:

```sh
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore="/var/jenkins_home/.ssl/keystore.jks" --httpsKeyStorePassword="5785ch00"
```

This will allow access through port 8443 while disabling HTTP for security reasons.



## **Step 2: SSH Connections in Jenkins**

Jenkins in the AdventureTube project requires **two separate SSH connections**:

### **1. Controller-Agent Connection**

This allows the Jenkins controller (master) to securely communicate with the agent over SSH. The controller orchestrates the build and deploy processes, while the agent handles actual execution, like testing and building.

- **Jenkins Master:** Initiates the connection (acts as the SSH client).
- **Jenkins Agent:** Receives the connection (acts as the SSH server).

#### **Step 1: Generate SSH Key Pair**

1. Generate an SSH key pair and register it for both master and agent (reference: [Jenkins SSH Agent Setup](https://www.jenkins.io/doc/book/using/using-agents/)).
   ```sh
   ssh-keygen -t ed25519 -C "jenkins-agent"
   ```
2. Name the private key `jenkins_agent_key`.
3. Register the private key in Jenkins Master as a credential.
4. Set the public key as an environment variable in your Docker Compose file for the agent.
5. During agent container creation, this public key will be added to the `known_hosts` file.

   **Note:** Ensure the private key has an extra carriage return at the end when uploaded to GitHub.

#### **Step 2: Configure Jenkins Agent Node**

- **Remote root directory:** `/home/jenkins/agent`
- **Launch method:** `Launch agents via SSH`
- **Host:** `localhost` or `IP address`
- **Port:** `2222`
- **Credentials:** `jenkins-agent-credential`
- **Host Key Verification Strategy:** Choose one of the following:
  1. **None verifying verification strategy** (not secure).
  2. **Known host verification strategy** (generate `known_hosts` on Jenkins Master using `ssh-keyscan` and change file permission to jenkins after creating the file. This way Jenkins master can securely connect to the agent without manual prompts or password authentication).
  3. **Manually trusted key verification strategy** (recommended for security).

    <p align="center"> <img src="/images/Hostkey verification Strategy .jpg"></p>

---

### **2. Git Repository SSH Connections**

The agent or master needs a second SSH connection to access the Git repository. When code is pushed to Git, Jenkins receives a notification (via a webhook), and the agent initiates the process of pulling the code from the repository for testing, building, and deployment.

Use the `id_ed25519` private key for communication with GitHub. Ensure this key is available on both Jenkins Agent and Master. Be cautious about any missing carriage returns in the private key content to prevent authorization errors.

---

## **Docker in Docker Issue on Jenkins Agent**

Jenkins Agent may encounter permission issues when accessing `/var/run/docker.sock`.
Although the Dockerfile for Jenkins Agent adds the Jenkins user to the Docker group within the agent container, it doesn't affect the Docker group on the host machine successfully.

This happens because the existing Docker group in the **jenkins-agent** base image may have a different **group ID** than the Docker group on the host machine. To resolve this, the Dockerfile must check for the existing Docker user, delete it, and create a new one with the same **Docker group ID** as the host machine.

To resolve this, follow these steps:

1. Edit the Docker service file:
   ```sh
   sudo nano /usr/lib/systemd/system/docker.service
   ```
2. Append the following lines to the bottom of the **Service** section:
   ```sh
   SupplementaryGroups=docker
   ExecStartPost=/bin/chmod 666 /var/run/docker.sock
   ```
3. Restart the Docker service:
   ```sh
   sudo service docker restart
   ```

**⚠️ Important:** Changing permissions to `666` for `/var/run/docker.sock` is **not recommended** for security reasons. The provided solution is a safer alternative to grant the necessary permissions.

By following these steps, you should be able to set up Jenkins with Docker Compose and resolve common **SSH and Docker-related issues**.

---

## **Conclusion**

The **AdventureTube microservices backend** is a **scalable, modular, and highly automated** system using **Jenkins, Docker, and Spring Cloud**. By leveraging an optimized **CI/CD pipeline**, the deployment process is **fast, reliable, and secure**.

🚀 **AdventureTube Deployment - Fully Automated & Scalable!**

