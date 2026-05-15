DO:
----


**1.Create** `dockerfile`

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```

**2.Build**
```bash
mvn clean package
docker build -t my-app .
```

**3.Run app by docker**
```bash
docker run -p 8080:8080 my-app
```
**4.Update** `docker-compose.yml` **file** ->
```bash
docker compose up --build
```

Flow:
`Code → Git → Jenkins → build jar → build Docker image → run container`

**5. Setup Jenkins**

5.1 gen password: `docker exec -it <jenkins-container-id> cat /var/jenkins_home/secrets/initialAdminPassword`

```text
docker exec -it f8cac1f2995f644f3b718534bdfbc44b4375186a62decb80e20f7da46e9f58a4 cat /var/jenkins_home/secrets/initialAdminPassword
```
<!-- 021a410179904f85b5374ca208d080f0 -->


5.2 create pipeline

```pipline
pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-app"
    }

    stages {
        stage('Fix Git Safe Dir') {
            steps {
                sh '''
                git config --global --add safe.directory /var/jenkins_home/workspace/my-app-pipeline
                '''
            }
        }

        stage('Clone') {
            steps {
                git 'https://github.com/DongNguyen20/i-deploy.git'
            }
        }
        
        stage('Debug') {
            steps {
                sh '''
                echo "ROOT:"
                ls -la
        
                echo "MYAPP:"
                ls -la myapp || true
                '''
            }
        }

        stage('Build JAR') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-17'
                    reuseNode true
                }
            }
            steps {
                dir('myapp') {
                    sh 'ls -la'   // debug
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME ./myapp'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop my-app-container || true
                docker rm my-app-container || true

                docker run -d \
                  -p 8083:8080 \
                  --name my-app-container \
                  $IMAGE_NAME
                '''
            }
        }
    }
}
```



