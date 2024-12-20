pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
       SCANNER_HOME = tool 'sonar-scanner'
       //dockerhub_cred=credentials('docker-cred')
       DOCKER_IMAGE = "greeshmab/boardgame:${env.BUILD_NUMBER}" // Image name with versioning
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Greeshma-Babu-tech/BoardGame.git'
            }
        }
        stage('Versioning') {
    steps {
        script {
            sh "mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}"
        }
    }
}
        stage('Compile') {
            steps {
                sh 'mvn compile'
                echo 'GITHUB Webhook Configured'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table --output trivy-fs-report.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. -Dsonar.exclusions=**/trivy-image-report.html'''
                }
            }
        }
        /*stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOUR') {
                        waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
                    }
                }
            }
        }*/
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Publish To Nexus') {
            steps { 
                nexusArtifactUploader(
                     nexusVersion: 'nexus3', // Specify the Nexus version 
                     protocol: 'http', 
                     nexusUrl: '35.175.171.59:8081', 
                     repository: 'BoardGame', 
                     credentialsId: 'nexus', 
                     groupId: 'com.javaproject', 
                     version: '0.0.1',
                     artifacts: [
                         [artifactId:'database_service_project',classifier: '', file: "target/database_service_project-1.0.${env.BUILD_NUMBER}.jar", type: 'jar']
                     ]
                    )
            withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy -X'
                }

            }
        }
        // stage('Build & Tag Docker Image') {
        //     steps {
        //         script {
        //             withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
        //                 sh 'docker build -t bkrraj/boardshack:latest .'
        //             }
        //         }
        //     }
        // }
        stage('Build Docker Image and TAG') {
            steps {
                script {
                    // Build the Docker image using the renamed JAR file
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t ${DOCKER_IMAGE} --build-arg JAR_FILE=target/app-${env.BUILD_NUMBER}.jar ."
                   }
                }   
            }
        } 
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE}"
            }
        }
        stage('Archive Report') {
            steps {
                // Archive the Trivy report for later reference
                archiveArtifacts artifacts: 'trivy-image-report.html', fingerprint: true
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }
        stage('Deploy To Kubernetes') { 
           steps { 
                /* withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                     withKubeConfig(caCertificate: '', clusterName: 'my-cluster', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://6EA25F755AE30BB5DF2E60A52FF787DA.sk1.us-east-1.eks.amazonaws.com') {
                         sh "aws eks --region us-east-1 update-kubeconfig --name my-cluster \
                             kubectl apply -f deployment-service.yaml"
                    } */
                echo "docker image deployed"
                }
                
        }
    }
}

