pipeline {
    agent any

    environment {
        DOCKER_USER = credentials('docker-user')
        DOCKER_PASSWORD = credentials('docker-password')
    }

    stages {
        stage('Setup') {
            steps {
                sh 'pip3 install -r ./etc/docker/flask/requirements-ci.txt'
            }
        }

        stage('Linting') {
            steps {
                sh 'make lint'
            }
        }

        stage('Security Testing') {
            steps {
                aquaMicroscanner imageName: 'alpine:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
            }
        }

        stage('Performace Testing') {
            steps {
                sh 'make performance-test'
            }
        }

        stage('General Testing') {
            steps {
                sh 'make test'
                sh 'make test-artifacts'
            }
        }

        stage('Build') {
            steps {
                sh 'make build'
            }
        }

        stage('Publish') {
            steps {
                sh 'make publish'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-static', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh """
                        mkdir -p ~/.aws
                        echo "[default]" >~/.aws/credentials
                        echo "[default]" >~/.boto
                        echo "aws_access_key_id = ${AWS_ACCESS_KEY_ID}" >>~/.boto
                        echo "aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}">>~/.boto
                        echo "aws_access_key_id = ${AWS_ACCESS_KEY_ID}" >>~/.aws/credentials
                        echo "aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}">>~/.aws/credentials
                    """
                }
                ansiblePlaybook playbook: 'infra/main.yml', inventory: 'infra/inventory'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'reports/web/**/*', allowEmptyArchive: true, fingerprint: true
            junit 'reports/junit/**/*.xml'
        }
    }
}
