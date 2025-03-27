pipeline {
    agent any

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/paingthu555/to-do-app.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'docker build -t getting-started-app .'
                    sh 'docker stop node || true'
                    sh 'docker rm node || true'
                    sh 'docker run -d --name node getting-started-app'
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'docker exec node npm test'
            }
        }

        stage('Create Image') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_LOGIN_PSW | docker login -u $DOCKERHUB_LOGIN_USR --password-stdin'
                    sh 'docker commit node paingthu555/node'
                    sh "docker push paingthu555/node"
                }
            }
        }

        stage('Deployment') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-server', passwordVariable: 'SERVER_PASSWORD', usernameVariable: 'SERVER_USERNAME')]) {
                        def remote = [:]
                        remote.name = 'docker'
                        remote.host = '54.179.58.202'
                        remote.user = "${SERVER_USERNAME}"
                        remote.password = "${SERVER_PASSWORD}"
                        remote.allowAnyHosts = true

                        sshCommand remote: remote, command: "echo $DOCKERHUB_LOGIN_PSW | docker login -u $DOCKERHUB_LOGIN_USR --password-stdin"
                        sshCommand remote: remote, command: "docker pull paingthu555/node"

                        sshCommand remote: remote, command: "docker stop node || true"
                        sshCommand remote: remote, command: "docker rm node || true"
                        sshCommand remote: remote, command: "docker run -dp 3000:3000 --name node paingthu555/node"
                    }
                }
            }
        }
    }

    post { 
        always { 
            sh 'docker stop node || true'
            sh 'docker system prune -fa'
        }
    }
}

