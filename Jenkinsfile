pipeline {
    environment {
        registry = "yasminelakhdher/user_app"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from GitHub
                    git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/yasmine-lakhdher/FrontEndUserApp.git'
                }
            }
        }

        stage('Build Angular App') {
            steps {
                script {
                    // Use the Node.js and npm installation defined in Jenkins configuration
                    def nodejsInstallation = tool name: 'NodeJS 14', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
                    env.PATH = "${nodejsInstallation}/bin:${env.PATH}"
                    // Change to the Angular app directory
                    dir('angular-app') {
                        // Install dependencies and node modules
                        sh 'npm install'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar4'
                PROJECT_NAME = "FrontEndUserApp"
            }
            steps {
               // git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/yasmine-lakhdher/FrontEndUserApp.git'
                
                withSonarQubeEnv(credentialsId: 'SonarCredentials',installationName: 'SonarServer') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=$PROJECT_NAME \
                        -Dsonar.sources=. """
                    
                }

                
            }
        }
        stage('Upload frontend application to nexus') {
            steps {
                
                sh "tar -czvf frontapp.tar.gz ./angular-app"
                nexusArtifactUploader artifacts: [[artifactId: 'frontapp', classifier: '', file: 'frontapp.tar.gz', type: 'tar.gz']], credentialsId: 'nexusCredentials', groupId: 'frontapp', nexusUrl: 'nexus:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'FrontendApplication', version: "1.${env.BUILD_NUMBER}.0"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dir('angular-app') {
                        sh "docker build -t yasminelakhdher/user_app:frontapp-${env.BUILD_NUMBER} ."
                        
                        
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            /*environment {
                DOCKERHUB_CREDENTIALS= 'docker-hub-yasmine'
            }*/
            steps {
                script {
               
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sh "echo \${DOCKERHUB_PASSWORD} | docker login -u \${DOCKERHUB_USERNAME} --password-stdin"
                        // Add the docker push step here
                        docker.withRegistry("https://registry.hub.docker.com", 'dockerhub') {
                            docker.image("${DOCKERHUB_USERNAME}/user_app:frontapp-${env.BUILD_NUMBER}").push()
                        }
                    }
                }
            }
            
        }
        stage('Deploy with Docker compose') {
            steps {
                script {
                    // Deploy the Docker image using docker-compose
                    sh "docker-compose up -d"
                }
            }
        }
        
            
    }
            


    

    post {
        always {  
    	    sh 'docker logout'     
        } 
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
