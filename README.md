# Flask Application with Jenkins CI/CD Pipeline

This project demonstrates a complete CI/CD pipeline using Jenkins to build, test, and deploy a Flask application using Docker containers.

## Project Structure

```
.
├── app/
│   └── main.py              # Flask application
├── tests/
│   └── test_app.py          # Application tests
├── Jenkinsfile              # Jenkins pipeline configuration
├── Dockerfile               # Application Dockerfile
├── jenkins.Dockerfile       # Jenkins container configuration
├── docker-compose.jenkins.yml       # Jenkins compose configuration
├── docker-compose.jenkins.wsl.yml   # Jenkins WSL compose configuration
├── docker-compose.dind.yml          # Docker-in-Docker compose configuration
├── requirements.txt         # Python dependencies
└── README.md               # This file
```

## Prerequisites

- Docker and Docker Compose installed
- Jenkins server (can be run using provided Docker Compose files)
- Docker Hub account
- Git

## Local Development

1. Create a Python virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run tests:
```bash
python -m pytest tests/
```

4. Run the application locally:
```bash
python app/main.py
```

The application will be available at `http://localhost:5000`

## Docker Setup

1. Build the application image:
```bash
docker build -t vrspi/flask-app:latest .
```

2. Run the container:
```bash
docker run -p 5000:5000 vrspi/flask-app:latest
```

## Jenkins Pipeline Setup

### 1. Jenkins Installation

Choose one of the following methods to run Jenkins:

#### Standard Docker Compose:
```bash
docker-compose -f docker-compose.jenkins.yml up -d
```

#### WSL Docker Compose:
```bash
docker-compose -f docker-compose.jenkins.wsl.yml up -d
```

#### Docker-in-Docker (DinD):
```bash
docker-compose -f docker-compose.dind.yml up -d
```

### 2. Initial Jenkins Configuration

1. Get the initial admin password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

2. Access Jenkins at `http://localhost:8080`

3. Install recommended plugins

4. Create admin user

### 3. Docker Hub Configuration

1. Create Docker Hub Access Token:
   - Log in to Docker Hub
   - Go to Account Settings > Security
   - Click "New Access Token"
   - Set appropriate permissions (Read, Write, Delete)
   - Save the token securely

2. Configure Jenkins Credentials:
   - Go to "Manage Jenkins" > "Manage Credentials"
   - Click "Jenkins" under Stores scoped to Jenkins
   - Click "Global credentials"
   - Click "Add Credentials"
   - Fill in:
     - Kind: Username with password
     - Username: Your Docker Hub username
     - Password: Your Docker Hub access token
     - ID: docker-hub-credentials
     - Description: Docker Hub Access Token

### 4. Pipeline Configuration

1. Create New Pipeline:
   - Click "New Item"
   - Enter name
   - Select "Pipeline"
   - Click OK

2. Configure Pipeline:
   - Under "Pipeline":
     - Definition: Pipeline script from SCM
     - SCM: Git
     - Repository URL: Your repository URL
     - Branch Specifier: */main
     - Script Path: Jenkinsfile

3. Save the configuration

## Pipeline Stages

1. **Checkout**: Clones the repository
2. **Test**: Runs Python tests
3. **Build Image**: Creates Docker image
4. **Push Image**: Pushes to Docker Hub
5. **Deploy**: Placeholder for deployment configuration

## Environment Variables

The following environment variables are configured in the Jenkinsfile:

- `DOCKER_IMAGE`: Docker image name (vrspi/flask-app)
- `DOCKER_TAG`: Build number
- `DOCKER_CREDENTIALS`: Jenkins credential ID for Docker Hub

## Troubleshooting

1. Docker Login Issues:
   - Verify Docker Hub credentials in Jenkins
   - Check access token permissions
   - Ensure credential ID matches Jenkinsfile

2. Build Failures:
   - Check Jenkins console output
   - Verify Docker daemon is running
   - Check network connectivity

3. Test Failures:
   - Review test results in Jenkins
   - Check test logs
   - Verify Python dependencies

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License. 