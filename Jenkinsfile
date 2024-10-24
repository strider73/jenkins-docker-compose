pipeline {
    agent any

    stages {
        stage('Verify tooling') {
            steps {
                sh '''
                docker version
                docker info
                docker compose version
                curl --version
                jq --version
                '''
            }
        }
        stage('Clone Repository') {
            steps {
                // Pull the latest code from GitHub
                git branch: 'main', url: 'git@github.com:strider73/jenkins-docker-compose.git'
            }
        }

        stage('Build Package') {
            steps {
                script {
                    // Use Maven wrapper to build the package
                    sh './mvnw package -DskipTests'
                }
            }
        }


        stage('Prune Docker Data') {
            steps {
                sh 'docker system prune -a --volumes -f'
            }
        }

        stage('Start container') {
            steps {
                sh 'docker compose up -d --no-color --wait'
                sh 'docker compose ps'
            }
        }

        // stage('Run test against the container') {
        //     steps {
        //         sh 'curl http://localhost:3000/param?query=demo | jq'
        //     }
        // }
    }

    post {
        always {
            sh 'docker compose down --remove-orphans -v'
            sh 'docker compose ps'
        }
    }
}
