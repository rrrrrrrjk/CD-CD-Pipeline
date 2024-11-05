pipeline {
    environment {
        registry = "rywj/jenkins-react"
        registryCredential = 'docker-token'
        dockerImage = ''
    }
    agent any
    stages{
        stage('Clear work station') {
            steps {
                cleanWs()
            }
        }
        stage('SCM') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/rrrrrrrjk/jenkins-react.git'
            }
        }
        /*stage('Quality Gate') {
            steps {
                script {
                    docker.image('sonarsource/sonar-scanner-cli').inside("--network Jenkins -v ${pwd()}:/usr/src") {
                        sh 'sonar-scanner'
                    }
                    sh 'docker run --rm --network Jenkins -v "C:/cicd pipelines/jenkins_home/workspace/image-project:/usr/src" sonarsource/sonar-scanner-cli sonar-scanner'
                }
            }
        }*/
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Building Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                // Run Trivy to scan the Docker image
                def trivyOutput = sh(script: "trivy image $dockerImage:$BUILD_NUMBER", returnStdout: true).trim()

                // Display Trivy scan results
                println trivyOutput

                // Check if vulnerabilities were found
                if (trivyOutput.contains("Total: 0")) {
                    echo "No vulnerabilities found in the Docker image."
                } else {
                    echo "Vulnerabilities found in the Docker image."
                    // You can take further actions here based on your requirements
                    // For example, failing the build if vulnerabilities are found
                    // error "Vulnerabilities found in the Docker image."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Trigger ManifestUpdate') {
            echo "triggering update-manifestjob"
            build job: 'update-manifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
        stage('Cleaning up') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }
    }
}