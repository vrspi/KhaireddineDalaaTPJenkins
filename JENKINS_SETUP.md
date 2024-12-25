# Jenkins Setup and Pipeline Configuration Guide

## 1. Installing Jenkins with Docker

### Prerequisites
- Docker and Docker Compose installed
- At least 2GB of RAM
- At least 10GB of disk space

### Docker Installation Steps

1. Make sure Docker and Docker Compose are installed:
```bash
docker --version
docker-compose --version
```

2. Create and start Jenkins container:
```bash
docker-compose -f docker-compose.jenkins.yml up -d
```

3. Get the initial admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

4. Access Jenkins:
- Open your browser and navigate to `http://localhost:8080`
- Use the initial admin password from step 3

### Installing Docker in Jenkins Container

The Docker socket and binary are already mounted in the container through the docker-compose configuration. However, you need to install some additional dependencies inside the Jenkins container:

1. Access the Jenkins container:
```bash
docker exec -it jenkins bash
```

2. Install Docker dependencies:
```bash
apt-get update && \
apt-get -y install apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common
```

3. Verify Docker works inside Jenkins:
```bash
docker ps
```

## 2. Initial Jenkins Configuration

1. Install suggested plugins when prompted
2. Create admin user
3. Configure Jenkins URL

## 3. Pipeline Configuration

### Required Plugins
1. Go to "Manage Jenkins" > "Manage Plugins"
2. Install the following plugins:
   - Docker Pipeline
   - Docker plugin
   - Git plugin
   - Pipeline
   - Pipeline: GitHub Groovy Libraries
   - Blue Ocean (optional but recommended)

### Configure Docker Hub Credentials

1. Go to "Manage Jenkins" > "Manage Credentials"
2. Click on "Jenkins" under Stores scoped to Jenkins
3. Click on "Global credentials"
4. Click "Add Credentials"
5. Select "Username with password"
6. Fill in your Docker Hub credentials
7. Set ID as 'docker-hub-credentials'

### Configure Docker Hub Access Token and Credentials

1. Create Docker Hub Access Token:
   - Log in to your Docker Hub account at https://hub.docker.com
   - Go to Account Settings (your username in top right) > Security
   - Click "New Access Token"
   - Name: "jenkins-pipeline" (or any meaningful name)
   - Access permissions: "Read & Write"
   - Click "Generate"
   - **IMPORTANT**: Copy the token immediately - it won't be shown again!

2. Add Token to Jenkins:
   - Go to "Manage Jenkins" > "Manage Credentials"
   - Click on "Jenkins" under Stores scoped to Jenkins
   - Click on "Global credentials (unrestricted)"
   - Click "Add Credentials"
   - Fill in the following EXACTLY:
     - Kind: Select "Username with password"
     - Scope: Select "Global (Jenkins, nodes, items, all child items, etc)"
     - Username: Your Docker Hub username (e.g., "vrspi")
     - Password: Paste your Docker Hub Access Token
     - ID: `docker-hub-credentials` (must match exactly with Jenkinsfile)
     - Description: "Docker Hub Access Token for Pipeline"
   - Click "Create"

3. Verify Credential Format:
   - The credential ID must be exactly `docker-hub-credentials`
   - Username should be your Docker Hub username (same as in DOCKER_IMAGE in Jenkinsfile)
   - Password should be the access token, not your Docker Hub password
   - Example:
     ```
     Kind: Username with password
     Username: vrspi
     Password: dckr_pat_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
     ID: docker-hub-credentials
     ```

### Creating the Pipeline

1. Click "New Item" on Jenkins dashboard
2. Enter a name for your pipeline
3. Select "Pipeline" and click OK
4. In the configuration:
   - Under "Pipeline":
     - Select "Pipeline script from SCM"
     - Select "Git" as SCM
     - Enter your repository URL
     - Specify the branch (e.g., */main)
     - Script Path: "Jenkinsfile"
5. Click "Save"

## 4. Running the Pipeline

1. Click "Build Now" to run the pipeline
2. View the progress in the Stage View
3. Check console output for any issues

## 5. Pipeline Stages Explanation

1. **Checkout**: Clones the repository
2. **Install Dependencies**: Installs Python requirements
3. **Run Tests**: Executes pytest suite
4. **Build Docker Image**: Creates Docker image from Dockerfile
5. **Push Docker Image**: Pushes image to Docker Hub
6. **Deploy**: Deploys application (customize as needed)

## 6. Troubleshooting

### Common Issues:
1. **Docker Permission Issues**:
   ```bash
   # Inside Jenkins container
   groupadd -f docker
   usermod -aG docker jenkins
   # Restart Jenkins container
   docker restart jenkins
   ```

2. **Container Access Issues**:
   ```bash
   # Check container logs
   docker logs jenkins
   
   # Verify volume permissions
   docker exec jenkins ls -la /var/jenkins_home
   ```

3. **Pipeline Syntax Errors**:
   - Use the Pipeline Syntax Generator
   - Check Jenkins logs

## 7. Best Practices

1. Always use version control
2. Keep sensitive information in Jenkins credentials
3. Use Jenkinsfile for pipeline configuration
4. Implement proper error handling
5. Use meaningful stage names
6. Add appropriate notifications
7. Regularly backup Jenkins home directory:
   ```bash
   # Backup
   docker run --rm --volumes-from jenkins -v $(pwd):/backup ubuntu tar cvf /backup/jenkins-backup.tar /var/jenkins_home
   ```

## 8. Docker-specific Notes

1. The Jenkins container has access to the host's Docker daemon through the mounted socket
2. Container-to-container networking is available through the `jenkins_network`
3. Jenkins home directory is persisted through a named volume
4. The container runs with root privileges to ensure Docker access
5. Port 8080 is exposed for web interface
6. Port 50000 is exposed for agent connections 