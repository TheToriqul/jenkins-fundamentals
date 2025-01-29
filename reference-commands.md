# Jenkins Infrastructure Command Reference Guide

### Project content table
- [Section 1: Core System Setup](#section-1-core-system-setup)
- [Section 2: AWS Infrastructure Management](#section-2-aws-infrastructure-management)
- [Section 3: Jenkins Job Operations](#section-3-jenkins-job-operations)
- [Section 4: Agent Configuration](#section-4-agent-configuration)
- [Section 5: Maintenance Guide](#section-5-maintenance-guide)

> **Author**: [Md Toriqul Islam](https://linkedin.com/in/thetoriqul)  
> **Description**: Complete command reference for Jenkins infrastructure setup and management  
> **Learning Focus**: Jenkins administration, AWS infrastructure, automation, and distributed builds  
> **Note**: Review and understand each command before execution in production environments

## Section 1: Core System Setup

### Step 1: System Preparation
```bash
# Update system packages
sudo apt -y update
sudo apt upgrade -y

# Install required dependencies
sudo apt install -y openjdk-17-jre-headless python3.8-venv curl git

# Verification command
java --version
python3 --version
```

### Step 2: Jenkins Repository Configuration
```bash
# Add Jenkins GPG key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add Jenkins repository
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null

# Verification command
cat /etc/apt/sources.list.d/jenkins.list
```

### Step 3: Jenkins Installation
```bash
# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start and enable service
sudo systemctl enable jenkins
sudo systemctl start jenkins

# Verification command
sudo systemctl status jenkins
```

### Step 4: Port Configuration
```bash
# Check current port usage
sudo netstat -tuln | grep 8080

# Update Jenkins port
sudo sed -i 's/HTTP_PORT=8080/HTTP_PORT=8081/' /etc/default/jenkins
sudo sed -i 's/^JENKINS_ARGS=""/JENKINS_ARGS="--httpPort=8081"/' /etc/default/jenkins

# Update systemd configuration
sudo sed -i 's/^ExecStart=.*/ExecStart=\/usr\/bin\/jenkins --httpPort=8081/' \
    /lib/systemd/system/jenkins.service

# Reload and restart services
sudo systemctl daemon-reload
sudo systemctl restart jenkins

# Verification command
sudo netstat -tuln | grep 8081
```

## Section 2: AWS Infrastructure Management

### Step 1: Infrastructure Setup
```bash
# Initialize project
mkdir jenkins-infrastructure && cd jenkins-infrastructure
python3 -m venv venv
source venv/bin/activate
pip install pulumi pulumi-aws

# Create Pulumi project
pulumi new aws-python

# Verification command
pulumi stack ls
```

### Step 2: Security Group Configuration
```bash
# View security groups
aws ec2 describe-security-groups --group-ids sg-XXXXX

# Configure Jenkins master security group
aws ec2 authorize-security-group-ingress \
    --group-id sg-XXXXX \
    --protocol tcp \
    --port 8081 \
    --cidr 0.0.0.0/0

# Configure agent security group
aws ec2 authorize-security-group-ingress \
    --group-id sg-YYYYY \
    --protocol tcp \
    --port 50000 \
    --cidr 10.0.0.0/16

# Verification command
aws ec2 describe-security-group-rules --filters Name="group-id",Values="sg-XXXXX"
```

## Section 3: Jenkins Job Operations

### Step 1: Workspace Management
```bash
# Create workspace directory
sudo mkdir -p /var/lib/jenkins/workspace/new-job
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/new-job

# Clear workspace
rm -rf /var/lib/jenkins/workspace/job-name/*

# Verification command
ls -la /var/lib/jenkins/workspace/
```

### Step 2: Job Configuration
```bash
# Create new job
java -jar jenkins-cli.jar -s http://localhost:8081/ create-job new-job < config.xml

# Update existing job
java -jar jenkins-cli.jar -s http://localhost:8081/ update-job existing-job < updated-config.xml

# Verification command
java -jar jenkins-cli.jar -s http://localhost:8081/ get-job job-name
```

### Step 3: Build Operations
```bash
# Trigger build
java -jar jenkins-cli.jar -s http://localhost:8081/ build job-name

# Trigger parameterized build
curl -X POST "http://localhost:8081/job/job-name/buildWithParameters" \
    --user "user:token" \
    --data "parameter=value"

# Verification command
tail -f /var/lib/jenkins/jobs/job-name/builds/latest/log
```

## Section 4: Agent Configuration

### Step 1: SSH Key Setup
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "jenkins-agent"

# Configure authorized keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Verification command
ssh -i ~/.ssh/id_rsa ubuntu@agent-ip whoami
```

### Step 2: Agent Directory Setup
```bash
# Create agent directory
sudo mkdir -p /opt/jenkins
sudo chown -R ubuntu:ubuntu /opt/jenkins
sudo chmod 755 /opt/jenkins

# Create workspace directory
sudo mkdir -p /opt/jenkins/workspace
sudo chown -R ubuntu:ubuntu /opt/jenkins/workspace

# Verification command
ls -la /opt/jenkins
```

## Section 5: Maintenance Guide

### Step 1: Backup Operations
```bash
# Backup Jenkins home
sudo tar -czf jenkins_backup.tar.gz /var/lib/jenkins/

# Backup specific jobs
sudo tar -czf jenkins_jobs_backup.tar.gz /var/lib/jenkins/jobs/

# Verification command
ls -lh *backup.tar.gz
```

### Step 2: Service Management
```bash
# Control Jenkins service
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins

# View detailed logs
sudo journalctl -u jenkins.service -f

# Verification command
sudo systemctl status jenkins
```

## Learning Notes

1. Always verify service status after configuration changes
2. Maintain regular backups of Jenkins configuration
3. Use appropriate security groups and firewall rules
4. Monitor system resources regularly
5. Keep security patches and plugins updated

---

> 💡 **Best Practice**: Always use SSH key authentication for agent connections and maintain proper key permissions

> ⚠️ **Warning**: Never expose Jenkins master directly to the internet without proper security measures

> 📝 **Note**: Adjust memory and resource allocations based on workload requirements