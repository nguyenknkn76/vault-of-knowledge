# Guide: Setting Up a Jenkins Pipeline to Push Docker Images to AWS ECR

This guide provides a comprehensive step-by-step process for configuring a Jenkins pipeline to build a Docker image and push it to an Amazon Elastic Container Registry (ECR) repository.

## Prerequisites

Before you begin, ensure you have the following:

1.  A running Jenkins server.
2.  An AWS account.
3.  An IAM User in your AWS account with programmatic access (Access Key ID and Secret Access Key).
4.  An ECR repository created in your AWS account.

## Step 1: Configure Your Jenkins Server

Your Jenkins instance needs a plugin and AWS credentials to interact with ECR.

### 1.1 Install the "Amazon ECR" Plugin

This plugin simplifies authentication with ECR.

1.  Navigate to **Manage Jenkins** > **Plugins**.
2.  Select the **Available Plugins** tab.
3.  In the search bar, type `Amazon ECR`.
4.  Select the checkbox next to the plugin and click **Install without restart**.

### 1.2 Add AWS Credentials to Jenkins

Securely store your IAM user's credentials in Jenkins.

1.  Navigate to **Manage Jenkins** > **Credentials**.
2.  Under **Stores scoped to Jenkins**, click on the **(global)** domain.
3.  Click **Add Credentials** on the left sidebar.
4.  Fill out the form:
    *   **Kind**: Select **AWS Credentials**.
    *   **ID**: Enter `aws-ecr-credentials`. This ID must match what you use in the `Jenkinsfile`.
    *   **Description**: A helpful description like `AWS credentials for ECR push`.
    *   **AWS Access Key ID**: Your IAM user's Access Key ID.
    *   **AWS Secret Access Key**: Your IAM user's Secret Access Key.
5.  Click **Create**.

> **Note on IAM Permissions:** For this to work, your IAM user needs permissions to interact with ECR. The AWS-managed policy `AmazonEC2ContainerRegistryPowerUser` is a good starting point.

## Step 2: Create Your `Jenkinsfile`

This declarative `Jenkinsfile` defines the entire build, test, and push process. It includes stages to manually install Docker and the AWS CLI, which requires the Jenkins agent user to have `sudo` privileges.

Place this file named `Jenkinsfile` in the root of your project's repository.

```groovy
// Jenkinsfile
pipeline {
    // This pipeline assumes a Linux agent with sudo rights to install tools.
    // A better practice is to use a pre-configured agent with these tools already installed.
    agent { label 'linux' }

    environment {
        // --- IMPORTANT: CONFIGURE THESE VARIABLES ---
        AWS_REGION         = 'us-east-1' // 1. Replace with your AWS region
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials' // 2. This must match the credential ID in Jenkins
        ECR_IMAGE_URI      = "YOUR_AWS_ACCOUNT_ID.dkr.ecr.${AWS_REGION}.amazonaws.com/your-ecr-repo-name" // 3. Replace placeholders
        // ------------------------------------------
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Tools') {
            steps {
                sh '''
                # Install Docker if not present
                if ! [ -x "$(command -v docker)" ]; then
                    echo "Installing Docker..."
                    sudo apt-get update -y && sudo apt-get install -y docker.io
                else
                    echo "Docker is already installed."
                fi

                # Install AWS CLI if not present
                if ! [ -x "$(command -v aws)" ]; then
                    echo "Installing AWS CLI..."
                    sudo apt-get update -y && sudo apt-get install -y awscli
                else
                    echo "AWS CLI is already installed."
                fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${ECR_IMAGE_URI}:${BUILD_NUMBER}"
                    // Use 'sudo' because the jenkins user is not in the docker group by default
                    sh "sudo docker build -t ${ECR_IMAGE_URI}:${BUILD_NUMBER} -t ${ECR_IMAGE_URI}:latest ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                // Use AWS credentials stored in Jenkins to log in to ECR
                withCredentials([aws(credentialsId: AWS_CREDENTIALS_ID, region: AWS_REGION)]) {
                    // Log in to ECR
                    sh "aws ecr get-login-password --region ${AWS_REGION} | sudo docker login --username AWS --password-stdin ${ECR_IMAGE_URI}"
                    
                    // Push the images to the ECR repository
                    sh "sudo docker push ${ECR_IMAGE_URI}:${BUILD_NUMBER}"
                    sh "sudo docker push ${ECR_IMAGE_URI}:latest"
                }
            }
        }
    }

    post {
        always {
            // Always log out of ECR as a cleanup step
            sh "sudo docker logout ${ECR_IMAGE_URI}"
        }
    }
}
```

## Step 3: Run the Pipeline

Once the Jenkins configuration is done and the `Jenkinsfile` is in your repository, you can create and run the pipeline job. It will perform the following actions:
1.  Check out your code.
2.  Install Docker and the AWS CLI (if they are not already present on the agent).
3.  Build your Docker image and tag it for your ECR repository.
4.  Use the stored AWS credentials to log in to ECR.
5.  Push the image to your ECR repository with two tags: one for the build number and one as `latest`.
6.  Log out of ECR.
---
