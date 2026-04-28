## Jenkins Slave (Agent) Integration Guide
This project outlines the architectural setup for a Distributed Jenkins Environment. By offloading build tasks to a dedicated Slave (Agent), we preserve the resources of the 4GB RAM Master, ensuring the Jenkins UI and orchestration remain responsive.

### 🏗 Architecture Overview
Jenkins Master: Orchestration, UI, and Credential Management.

Jenkins Slave (172.31.47.211): Executes Maven builds, Unit Tests, and Java compilations.

Communication: Secured via SSH Key-based authentication.

### 📂 Setup Phases
Phase 1: Key Generation (Master Node)
Generate the identity for the jenkins user on the Master.

Log into Master as jenkins.

Execute:

ssh-keygen -t rsa -b 4096 -m PEM -f ~/.ssh/id_rsa_jenkins -N ""
Copy the public key: cat ~/.ssh/id_rsa_jenkins.pub.

Phase 2: Agent Preparation (Slave Node)
Prepare the remote environment to host the Jenkins agent process.

User Setup:

sudo useradd -m jenkins
sudo su - jenkins
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
Dependencies: Install Java (required for the agent .jar) and Maven.

Bash

sudo dnf install java-21-amazon-corretto-devel maven -y
Phase 3: Jenkins UI Configuration
Credentials: Add the Private Key (~/.ssh/id_rsa_jenkins) as an SSH Username with private key credential named slave-ssh-key.

Node Setup: Create Slave-01 as a Permanent Agent.

Remote root: /home/jenkins

Launch method: Launch agents via SSH.

Host Key Verification: Non-verifying Verification Strategy.

Threshold Optimization (SRE Best Practice): > Crucial: In the "Configure Monitors" section, adjust Free Disk Space and Free Temp Space thresholds from 1GiB to 200MiB. This prevents the agent from going offline on smaller t2.micro instances.

### 🚀 Validation Pipeline
Use the following declarative pipeline to verify that the Slave is correctly assuming the build workload.

pipeline {
    agent { label 'Slave-01' } 

    stages {
        stage('Identify Environment') {
            steps {
                echo "Running on Node: ${env.NODE_NAME}"
                sh 'hostname && uptime'
            }
        }

        stage('Check Build Tools') {
            steps {
                echo 'Verifying toolchains...'
                sh 'java -version && mvn -version'
            }
        }
    }
}
