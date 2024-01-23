pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '6f85d23c-26d9-4702-b591-62c5e6ac2ff9', poll: false, url: 'https://github.com/mdsyedali8/Jenkins-Ekart.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

        stage('OWASP Dependency-Check Vulnerabilities') {
            steps {
                script {
                    def dependencyCheckHome = tool 'DP'
                    def dependencyCheckScript = "${dependencyCheckHome}/bin/dependency-check.sh"
                    sh "${dependencyCheckScript} --scan ./"
                }
            }
        }

        stage('Publish Dependency-Check Report') {
            steps {
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('sonarqube') {
            steps {
                withSonarQubeEnv ('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=shopping-cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=shopping-cart
        '''
}

                
            }
        }
        
        stage('BUILD') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('DOCKER build & push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: '47adeae7-89a9-4088-91d7-a392dcf6cb33', toolName: 'docker') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart mdsyedali/shopping-cart:latest"
                        sh "docker push mdsyedali/shopping-cart:latest"
                        
                        
                    }
                }
            }
        }

    }
}
