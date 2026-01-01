### Phase 1: Infrastructure Setup (EC2 & Security)

First, we'll create the two EC2 instances and configure their security groups to allow them to communicate.

1.  **Create a Key Pair:**
    *   In the AWS EC2 Console, go to **Network & Security > Key Pairs**.
    *   Create a new key pair (e.g., `jenkins-key`). Download and save the `.pem` file. You will need it to SSH into your instances.

2.  **Create Security Groups:**
    *   Go to **Network & Security > Security Groups**.
    *   **Create `jenkins-master-sg`:**
        *   **Inbound Rules:**
            *   `SSH` (Port 22) from `My IP` (for your access).
            *   `HTTP` (Port 80) from `Anywhere` (or your IP for more security).
            *   `Custom TCP` (Port 8080) from `Anywhere` (for the Jenkins UI).
            *   `Custom TCP` (Port 50000) from the `jenkins-agent-sg` (for agent communication). You'll add the agent security group ID here after creating it.
    *   **Create `jenkins-agent-sg`:**
        *   **Inbound Rules:**
            *   `SSH` (Port 22) from `My IP`.
            *   Allow all traffic from `jenkins-master-sg`.

3.  **Launch EC2 Instances:**
    *   Go to **Instances > Launch Instances**.
    *   **Instance 1 (Jenkins Master):**
        *   **Name:** `Jenkins-Master`
        *   **AMI:** `Ubuntu 22.04 LTS` (or newer).
        *   **Instance Type:** `t3.medium` or `t2.medium` is recommended for a master.
        *   **Key Pair:** Select the `jenkins-key` you created.
        *   **Network Settings:** Assign the `jenkins-master-sg` security group.
    *   **Instance 2 (Jenkins Agent):**
        *   **Name:** `Jenkins-Agent`
        *   **AMI:** `Ubuntu 22.04 LTS`.
        *   **Instance Type:** `t3.small` or `t2.small` is sufficient.
        *   **Key Pair:** Select the `jenkins-key`.
        *   **Network Settings:** Assign the `jenkins-agent-sg` security group.

4.  **Update Master Security Group:**
    *   Go back to `jenkins-master-sg` and edit the inbound rule for Port 50000. Set the source to the ID of the `jenkins-agent-sg`.

---

### Phase 2: Jenkins Master Setup (Docker Compose)

We will install Docker and use Docker Compose to run the Jenkins master for easy management and data persistence.

1.  **Connect to your Jenkins Master instance:**
    ```bash
    ssh -i /path/to/your/jenkins-key.pem ubuntu@<MASTER_EC2_PUBLIC_IP>
    ```

2.  **Install Docker and Docker Compose:**
    ```bash
    # Update package list and install prerequisites
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl gnupg

    # Add Docker's official GPG key
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Set up the repository
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    
    # Install Docker Engine
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # Add your user to the docker group to run docker without sudo
    sudo usermod -aG docker $USER
    newgrp docker
    ```

3.  **Create the `docker-compose.yml` file:**
    *   Create a directory for your Jenkins setup and create the file:
        ```bash
        mkdir jenkins-master
        cd jenkins-master
        nano docker-compose.yml
        ```
    *   Paste the following content into the file. This configuration creates a persistent volume for your Jenkins data, which is a best practice.

        ```yaml
        version: '3.8'
        services:
          jenkins:
            image: jenkins/jenkins:lts-jdk17
            privileged: true
            user: root
            ports:
              - "8080:8080"
              - "50000:50000"
            container_name: jenkins
            volumes:
              - jenkins_home:/var/jenkins_home
              - /var/run/docker.sock:/var/run/docker.sock
        
        volumes:
          jenkins_home:
        ```

4.  **Start Jenkins:**
    ```bash
    docker compose up -d
    ```

5.  **Initial Jenkins Setup:**
    *   Navigate to `http://<MASTER_EC2_PUBLIC_IP>:8080`.
    *   Jenkins will ask for an initial admin password. Get it from the container's logs:
        ```bash
        docker logs jenkins
        ```
    *   Copy the password, paste it into the setup screen, and click **Continue**.
    *   On the next screen, choose **Install suggested plugins**.
    *   Create your first admin user and complete the setup.

---

### Phase 3: Jenkins Agent Setup

Now, we'll configure the second EC2 instance to act as a build agent where Docker commands will actually run.

1.  **Connect to your Jenkins Agent instance:**
    ```bash
    ssh -i /path/to/your/jenkins-key.pem ubuntu@<AGENT_EC2_PUBLIC_IP>
    ```

2.  **Install Java and Docker:**
    *   The Jenkins agent requires a Java Runtime Environment. Docker is needed to build your images.
    ```bash
    sudo apt-get update
    sudo apt-get install -y openjdk-17-jre
    # Follow the same Docker installation steps from Phase 2, Step 2
    ```

3.  **Create a Jenkins user and directory:**
    ```bash
    sudo useradd -m -s /bin/bash jenkins
    sudo su - jenkins
    mkdir .ssh
    # This directory will be the agent's workspace
    mkdir /home/jenkins/agent
    ```

4.  **Configure the Agent Node in Jenkins:**
    *   On your Jenkins dashboard, go to **Manage Jenkins > Nodes**.
    *   Click **New Node**.
    *   **Node Name:** `docker-agent-1`
    *   Select **Permanent Agent** and click **Create**.
    *   **Configuration:**
        *   **Remote root directory:** `/home/jenkins/agent`
        *   **Labels:** `docker linux` (This is important! It's how your pipeline will select this agent).
        *   **Usage:** `Use this node as much as possible`.
        *   **Launch method:** **Launch agent by connecting it to the controller**.
        *   Click **Save**.

5.  **Connect the Agent to the Master:**
    *   After saving, you'll be on the agent's status page. You will see instructions with a command like:
        `java -jar agent.jar -jnlpUrl http://<MASTER_IP>:8080/computer/docker-agent-1/slave-agent.jnlp -secret <SECRET_KEY> -workDir "/home/jenkins/agent"`
    *   On your **agent EC2 instance**, run these two commands to download the agent and connect:
        ```bash
        curl -sO http://<MASTER_EC2_PUBLIC_IP>:8080/jnlpJars/agent.jar
        # Run the command from the Jenkins UI
        java -jar agent.jar -jnlpUrl ... 
        ```
    *   Go back to the Jenkins **Nodes** page. You should see `docker-agent-1` is now connected.

    **(Best Practice)** For a production setup, you would run the agent as a `systemd` service so it starts automatically on boot.

---

### Phase 4: Jenkins Configuration (Credentials)

Securely store your DockerHub credentials in Jenkins.

1.  **Create a DockerHub Access Token:**
    *   Go to your DockerHub account settings > **Security**.
    *   Click **New Access Token**, give it a name (e.g., `jenkins-token`), and set permissions to **Read, Write, Delete**.
    *   Copy the generated token. **You will not see it again.**

2.  **Add Credentials to Jenkins:**
    *   In Jenkins, go to **Manage Jenkins > Credentials**.
    *   Under **Stores scoped to Jenkins**, click on the **(global)** domain.
    *   Click **Add Credentials**.
    *   **Kind:** `Username with password`.
    *   **Username:** Your DockerHub username.
    *   **Password:** The DockerHub access token you just created.
    *   **ID:** `dockerhub-credentials` (This ID will be used in your pipeline script).
    *   Click **Create**.

---

### Phase 5: Creating the Jenkins Pipeline Job

This job will listen for triggers from GitHub and run the pipeline defined in your repository's `Jenkinsfile`.

1.  On the Jenkins dashboard, click **New Item**.
2.  Enter a name (e.g., `react-app-pipeline`), select **Pipeline**, and click **OK**.
3.  **Pipeline Tab Configuration:**
    *   **Definition:** Select **Pipeline script from SCM**.
    *   **SCM:** Select **Git**.
    *   **Repository URL:** Enter the HTTPS URL of your GitHub repository (e.g., `https://github.com/your-username/your-repo.git`).
    *   **Branch Specifier:** `*/main` or `*/master`, depending on your default branch.
    *   **Script Path:** Leave it as `Jenkinsfile`. This is the file Jenkins will look for in your repo's root.
4.  **Build Triggers Tab:**
    *   Check the box for **GitHub hook trigger for GITScm polling**.
5.  Click **Save**.

---

### Phase 6: The `Jenkinsfile` for Your React Project

In the root of your ReactJS GitHub repository, create a file named `Jenkinsfile` (no extension) with the following content.

```groovy
pipeline {
    // 1. Select the agent with the 'docker' and 'linux' labels
    agent {
        label 'docker && linux'
    }

    environment {
        // 2. Define environment variables
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        // IMPORTANT: Use your actual DockerHub username and image name
        DOCKER_IMAGE_NAME = 'your-dockerhub-username/your-react-app' 
    }

    stages {
        stage('Checkout') {
            steps {
                // 3. Checkout the source code from the triggered commit
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // 4. Build the image and tag it with the build number and 'latest'
                    echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} -t ${DOCKER_IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                // 5. Use the credentials stored in Jenkins to log in and push
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        // 6. Always log out of DockerHub as a cleanup step
        always {
            sh "docker logout"
        }
    }
}
```

**Commit and push this `Jenkinsfile` and your project's `Dockerfile` to your GitHub repository.**

---

### Phase 7: GitHub Webhook Integration

The final step is to tell GitHub to notify Jenkins whenever there's a push or pull request.

1.  In your GitHub repository, go to **Settings > Webhooks**.
2.  Click **Add webhook**.
3.  **Configuration:**
    *   **Payload URL:** `http://<MASTER_EC2_PUBLIC_IP>:8080/github-webhook/` (Make sure to include the trailing slash).
    *   **Content type:** `application/json`.
    *   **Which events would you like to trigger this webhook?** Select **Send me everything** or choose **Let me select individual events** and check **Pushes** and **Pull requests**.
4.  Click **Add webhook**.

You should see a green checkmark next to the webhook if the connection to your Jenkins server was successful.

You are now all set! When you push a new commit or create a pull request in your GitHub repository, it will automatically trigger the Jenkins pipeline. The pipeline will use the dedicated agent to build your React app's Docker image and push it to DockerHub.

### Install AWS CLI

```bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Install Plugin `Amazon ECR`