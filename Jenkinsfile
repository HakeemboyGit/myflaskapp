pipeline {
    agent any

    environment {
        SERVER = 'deploy@192.168.1.4'
        APP_DIR = '/var/www/myflaskapp'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv build-venv
                    build-venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    if [ -d tests ]; then
                        build-venv/bin/pip install pytest
                        build-venv/bin/pytest tests/
                    fi
                '''
            }
        }

        stage('Deploy to Linux') {
            steps {
                sshagent(credentials: ['prod-ssh-key']) {

                    sh '''
                        rsync -az --delete \
                          --exclude='.git/' \
                          --exclude='build-venv/' \
                          --exclude='venv/' \
                          ./ $SERVER:$APP_DIR/

                        ssh $SERVER "
                            cd $APP_DIR &&
                            python3 -m venv venv &&
                            venv/bin/pip install -r requirements.txt &&
                            sudo systemctl restart myflaskapp &&
                            sudo systemctl is-active myflaskapp
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Flask deployment successful!'
        }

        failure {
            echo 'Flask deployment failed!'
        }
    }
}