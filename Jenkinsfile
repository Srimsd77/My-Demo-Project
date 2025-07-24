pipeline {
    agent { label 'Slave-1'}
    
    tools{
        maven 'maven'
    }
    environment {
        SONARQUBE_ENV = 'sonar'
        ECR_REPO = '253490751559.dkr.ecr.ap-south-1.amazonaws.com/devops'
        AWS_REGION = 'ap-south-1'
        IMAGE_NAME = 'my-img'
    }    
    stages {
        stage('Checkout') {
            agent { label 'Slave-1'} 
            steps {
                git 'https://github.com/Srimsd77/My-Demo-Project.git'
                
            }
        }
        stage('SonarQube Analysis') {
            agent { label 'Slave-1'} 
            steps {
                echo "Starting SonarQube static code analysis..."
                withSonarQubeEnv("${env.SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'token', variable: 'SONAR_TOKEN')]) {
                        sh """
                            mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=demo \
                              -Dsonar.projectName='demo' \
                              -Dsonar.host.url=http://13.204.82.179:9000 \
                              -Dsonar.token=$SONAR_TOKEN
                        """
                    }
                }
            }
        }        
        stage('Code-Build') {
            agent { label 'Slave-1'} 
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Docker Build & Push to ECR') {
            agent { label 'Slave-1'}             
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'demo') {
                        sh 'docker build -t $IMAGE_NAME .'
                        sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}"
                        sh 'docker tag $IMAGE_NAME:latest $ECR_REPO:latest'
                        sh 'docker push $ECR_REPO:latest'
                    }
                }
            }
        }        
       stage('Deploy') {
            agent { label 'Slave-1'}            
            steps {
                script {
                    echo "Building Docker image and pushing to ECR..."
                    withAWS(region: "${env.AWS_REGION}", credentials: 'demo') {
                       sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_REPO.split('/')[0]}"
                       sh 'docker pull $ECR_REPO:latest'
                       sh 'docker stop my-con'
                       sh 'docker rm my-con'
                       sh 'docker run -itd --name con-2 -p "81:8080" $ECR_REPO:latest '
                    }
                }
            }
        }               
    }
}
