pipeline {
    // agent any
    agent {
        docker {image 'jenkins/ssh-agent:jdk17'}
    }
    stages{
        stage('Verify tooling'){
            steps{
                sh'''
                docker version
                docker info
                docker compose version
                curl --version
                jp --vedocker rsion
                '''
            }
        }

        stage('Prune Docker Data'){
            steps{
                sh 'docker system prune -a --volumes -f'
            }
        }

        stage('Start container '){
            steps{
                sh 'docker compose up -d --no-color --wait'
                sh 'docker compose ps'
            }
        }

        stage('Run test against the container'){
            steps{
                sh 'curl http://localhost:3000/param?query=demo | jq '
            }
        }
    }
}

post {
    always {
        sh 'docker compose down --remove-orpans -v'
        sh 'docker compose ps'
    }
}