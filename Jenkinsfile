pipeline {
    agent {
        node {
            label "prod"
        }
    }
    stages {
        stage ("Code") {
            steps {
                git 'https://github.com/kakanivamsikrishna/python-code-library-app.git'
            }
        }
        stage ("CQA") {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv("mysonar") {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        stage ("Quality-Check") {
            steps {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
            }
        }
        stage ("Image-Creation") {
            steps {
                sh'''
                docker build -t vamsikrishnakakani/library-app:db database
                docker build -t vamsikrishnakakani/library-app:authentication auth
                docker build -t vamsikrishnakakani/library-app:books book
                docker build -t vamsikrishnakakani/library-app:borrow borrow
                docker build -t vamsikrishnakakani/library-app:frontend .
                '''
            }
        }
        stage ("Trivy-Scan") {
            steps {
                sh'''
                trivy image vamsikrishnakakani/library-app:db >> dbreport.txt
                trivy image vamsikrishnakakani/library-app:authentication >> authreport.txt
                trivy image vamsikrishnakakani/library-app:books >> bookreport.txt
                trivy image vamsikrishnakakani/library-app:borrow >> borrowreport.txt
                trivy image vamsikrishnakakani/library-app:frontend >> frontendreport.txt
                '''
            }
        }
        stage ("Registry-Dockerhub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub') {
                        sh'''
                        docker push vamsikrishnakakani/library-app:db
                        docker push vamsikrishnakakani/library-app:authentication
                        docker push vamsikrishnakakani/library-app:books
                        docker push vamsikrishnakakani/library-app:borrow
                        docker push vamsikrishnakakani/library-app:frontend
                        '''
                    }
                }
            }
        }
        stage ("Deploy-DockerStack") {
            steps {
                sh "docker stack deploy myapp --compose-file=compose.yml"
            }
        }
    }
}
