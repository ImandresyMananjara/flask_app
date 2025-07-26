pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent-my-app'
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
                labels:
                    component: ci
            spec:
                containers:
                - name: python
                  image: python:3.7
                  command:
                  - cat
                  tty: true
                - name: docker
                  image: docker
                  command:
                  - cat
                  tty: true
                  volumeMounts:
                  - mountPath: /var/run/docker.sock
                    name: docker-sock
                - name: kubectl
                  image: lachlanevenson/k8s-kubectl:v1.17.2
                  command:
                  - cat
                  tty: true
                volumes:
                - name: docker-sock
                  hostPath:
                    path: /var/run/docker.sock
            '''
        }
    }
    environment {
        GIT_CREDENTIALS_ID = 'github-token'
    }
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('Cloner le dépôt') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/ImandresyMananjara/flask_app.git', branch: 'main'
            }
        }
        stage('Lister les fichiers') {
            steps {
                container('python') {
                    sh 'ls -la'
                }
            }
        }
        stage('Tester l\'application') {
            steps {
                container('python') {
                    sh '''
                        python3 --version
                        pip3 install -r requirements.txt
                        python3 test.py
                    '''
                }
            }
        }
        stage('Build Image') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t localhost:4000/pythontest:latest .
                        docker push localhost:4000/pythontest:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f kubernetes/deployment.yaml -n jenkins
                        kubectl apply -f kubernetes/service.yaml -n jenkins
                    '''
                }
            }
        }
    }
}