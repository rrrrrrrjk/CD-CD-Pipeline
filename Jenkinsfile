pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        string(name: 'DOCKER_REGISTRY', defaultValue: 'rywj/jenkins-react', description: 'Docker registry')
    }
    environment {
        registry = "${DOCKER_REGISTRY}"
        registryCredential = 'docker-token'
        dockerImage = ''
    }
    tools {
        nodejs 'nodejs' 
    }
    stages {
        stage('Clear Workstation') {
            steps {
                cleanWs()
            }
        }
        stage('SCM') {
            steps {
                git branch: "${BRANCH_NAME}", credentialsId: 'github-token', url: 'https://github.com/rrrrrrrjk/jenkins-react.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Sonar Quality Gate') {
            steps {
                script {
                    docker.image('sonarsource/sonar-scanner-cli').inside("--network Jenkins -v ${pwd()}:/usr/src") {
                        sh 'sonar-scanner'
                    }
                }
            }
            post {
                failure {
                    error "SonarQube Quality Gate failed."
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy Security Scans') {
            steps {
                script {
                    // Filesystem Scan
                    echo "Running Trivy filesystem scan..."
                    sh "trivy fs . > trivyfs.txt"

                    // Docker Image Scan
                    echo "Running Trivy image scan..."
                    def trivyOutput = sh(script: "trivy image --quiet --ignore-unfixed ${dockerImage.imageName()}", returnStdout: true).trim()

                    // Display scan results
                    println trivyOutput

                    // Fail if vulnerabilities are found
                    if (!trivyOutput.contains("Total: 0")) {
                        error "Vulnerabilities found in the Docker image."
                    }
                }
            }
        }
        stage('Building Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Trigger Manifest Update') {
            steps {
                script {
                    echo "Triggering update-manifest job..."
                    build job: 'update-manifest'
                }
            }
        }
        stage('Cleaning Up') {
            steps {
                sh "docker rmi ${dockerImage.imageName()}"
            }
        }
        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'trivyfs.txt, dependency-check-report.xml', allowEmptyArchive: true
            }
        }
    }
}
