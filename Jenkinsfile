pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "rpdharanidhar/wordsmith"
        SONAR_LOGIN = "admin"
        SONAR_PASSWORD = "polar"
        SONAR_HOST_URL = 'http://168.138.184.191:9000/'
        TAG = "latest"
        CLAIR_SCANNER_VERSION = "latest"
        CLAIR_URL = "http://localhost:6060"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/rpdharanidhar/wordsmith.git', branch: 'main'
                // checkout scm for the dev branch
            }
        }

        stage('print the commit id') {
            steps {
                script {
                    // Print the commit ID
                    echo "Commit ID: ${env.GIT_COMMIT}"
                }
            }
        }

        stage('SonarQube-Analysis') {
            steps {
                script {
                    try {
                        def scannerHome = tool 'sonarqube-scanner';
                        withSonarQubeEnv("sonarqube-server") {
                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=Testing-NodeApp-Jest \
                                    -Dsonar.sources=. \
                                    -Dsonar.host.url=${SONAR_HOST_URL} \
                                    -Dsonar.login=${SONAR_LOGIN} \
                                    -Dsonar.password=${SONAR_PASSWORD}
                            """
                        }
                    } catch (Exception e) {
                        echo "SonarQube stage failed: ${e.message}"
                        error("Stopping pipeline due to SonarQube Analysis failure.")
                    }
                }
            }
        }

        stage('Install Dependencies for clair') {
            steps {
                script {
                    // Install dependencies
                    sh 'sudo apt install docker-compose -y'
                    sh 'sudo apt install docker'
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerHubCredentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh """
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            sudo docker compose up --build
                            sudo docker push ${DOCKER_IMAGE_NAME}
                        """
                    }
                }
            }
        }

        stage('Scan with Clair') {
            steps {
                script {
                    // Pull Clair Scanner Docker image
                    docker.image("ovotech/clair-scanner:${CLAIR_SCANNER_VERSION}").pull()

                    // Run Clair Scanner
                    // Step 1: Build the Docker image
                    sh 'sudo docker-compose -f /home/ubuntu/Testing-NodeApp-Jest/clair/docker-compose.yaml up -d'

                    // Step 2: Verify the image exists
                    sh 'docker images'

                    // Step 3: Run Clair Scanner
                    // sh 'sudo docker run --rm --net=host -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/tmp objectiflibre/clair-scanner:latest --clair=http://localhost:6060 --ip=localhost rpdharanidhar/testing-nodeapp-jest:latest'
                    sh 'mkdir -p /tmp/jenkins-workspace'
                    sh 'sudo chmod 777 /tmp/jenkins-workspace'
                    sh 'docker-compose up -d'
                    // sh 'docker run --rm --net=host -v /var/run/docker.sock:/var/run/docker.sock -v /tmp/jenkins-workspace:/tmp objectiflibre/clair-scanner:latest --clair=http://localhost:6060 --ip=localhost rpdharanidhar/testing-nodeapp-jest:latest'
                    sh 'docker run --rm --net=host -v /var/run/docker.sock:/var/run/docker.sock -v /tmp/jenkins-workspace:/tmp ovotech/clair-scanner:latest --clair=http://localhost:7070 --ip=localhost rpdharanidhar/testing-nodeapp-jest:latest'
                }
            }
        }       
    }


    post {
        always {
            script {
                echo "Pipeline completed."
            }
        }
        success {
            echo 'Deployment and tests were successful!'
        }
        failure {
            echo 'There were test failures.'
        }
    }
}
