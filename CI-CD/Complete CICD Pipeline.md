## **Complete CI/CD pipeline (Jenkins)**

**Objective:** Design and implement an end-to-end CI/CD pipeline in Jenkins that builds, tests, containerizes a simple web service, pushes the image to a registry, and deploys it to an EC2 host.

---

### **Instructions**

#### **Application:**

- Create a simple app (Node.js/Express or Python/Flask) with unit tests and a Dockerfile

#### **Jenkins setup:**

- Install required plugins: Pipeline, Git, Credentials Binding, Docker, SSH Agent

- Create the following Jenkins credentials:

  - git_credentials (optional)

  - registry_creds

  - ec2_ssh

#### **Pipeline stages:**

| **Stage** | **Description** |
| --- | --- |
| Checkout | Pull source code from GitHub |
| Install/Build | Install dependencies and build the application |
| Test | Run unit tests |
| Docker Build | Build the Docker image |
| Push Image | Push the image to the container registry |
| Deploy | SSH into EC2 and deploy the new image |

#### **Verification:**

- Confirm a successful pipeline run

- Verify the application is accessible via the EC2 public DNS or IP

- Clean up residual containers and images after deployment

---

### **Submission requirements**

| **Item** | **Format** | **Contents** |
| --- | --- | --- |
| GitHub repository | All project files | App source code, unit tests, Dockerfile, Jenkinsfile, runbook, screenshots and logs |
| Tools evidence | Version output or screenshot | Jenkins LTS, Docker, GitHub, EC2 (Amazon Linux 2) |
| Pipeline evidence | Screenshots | Successful pipeline run showing all stages (build → test → push → deploy) |
| App evidence | Screenshot | Accessible application via EC2 public DNS or IP |

---

### **Required file structure**

```
project/
├── app.py              # or app.js for Node.js
├── test_app.py         # or test_app.js for Node.js
├── requirements.txt    # or package.json for Node.js
├── Dockerfile
├── Jenkinsfile
├── runbook.md
└── screenshots/
    ├── pipeline_success.png
    ├── app_accessible.png
    ├── docker_push.png
    └── deploy_stage.png
```

---

### **Jenkinsfile pipeline structure**

```groovy
pipeline {
    agent any

    environment {
        REGISTRY_CREDS = credentials('registry_creds')
        EC2_SSH        = credentials('ec2_ssh')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Install / Build') {
            steps {
                sh 'pip install -r requirements.txt'   // or: npm install
            }
        }

        stage('Test') {
            steps {
                sh 'pytest test_app.py'                // or: npm test
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t <your-image-name>:latest .'
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    echo $REGISTRY_CREDS_PSW | docker login -u $REGISTRY_CREDS_USR --password-stdin
                    docker push <your-image-name>:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2_ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@<EC2-PUBLIC-IP> "
                            docker pull <your-image-name>:latest &&
                            docker stop app || true &&
                            docker rm app || true &&
                            docker run -d --name app -p 5000:5000 <your-image-name>:latest
                        "
                    '''
                }
            }
        }

    }

    post {
        always {
            sh 'docker image prune -f'
        }
    }
}
```

---

### **Tips**

- Store all credentials in Jenkins Credentials Manager — never hardcode secrets in your Jenkinsfile

- Use || true after docker stop and docker rm to prevent the pipeline from failing if no container is running

- Add a post { always { } } block to clean up dangling images after every run

- Test your SSH connection to EC2 manually before running the pipeline to confirm key-based authentication works

- Enable StrictHostKeyChecking=no in your SSH command only for lab environments — use known hosts in production

- Tag your Docker images with the Jenkins build number (e.g., image:${BUILD_NUMBER}) for traceability alongside the latest tag
