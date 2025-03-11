pipeline {
    environment {
        IMAGEN = "juanantpineda/flask-app"
        USUARIO = 'USER_DOCKERHUB'
    }
    agent none
    stages {
        stage("test the project") {
            agent {
                docker {
                image "python:3"
                args '-u root:root'
                }
            }
            stages {
                stage('Clonando el repositorio') {
                    steps {
                        git branch:'main',url:'https://github.com/JuanantPineda/flask-app.git'
                    }
                }
                stage('Instalacion del requirements.txt') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'cd app && pytest test_app.py'
                    }
                }
            }
        }
        stage('Creación de la imagen') {
            agent any
            stages {
                stage('Creando la imagen') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        script {
                            docker.withRegistry( '', USUARIO ) {
                                newApp.push()
                                newApp.push("latest")
                            }
                        }
                    }
                }
                stage('Clean Up') {
                    steps {
                        sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                        }
                }
            }
        }
        stage ('Despliegue') {
            agent any
            stages {
                stage ('Despliegue django_publicaciones'){
                    steps{
                        sshagent(credentials : ['SSH_KEY']) {
                        sh 'ssh -o StrictHostKeyChecking=no debian@luffy.juanpiece.es "cd flask-app && git pull && docker-compose down && docker pull juanantpineda/flask-app:latest && docker-compose up -d"'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
        mail to: 'juanantpiama@gmail.com',
        subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
        body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }
}