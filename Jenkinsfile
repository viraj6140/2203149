pipeline {
    agent any

    environment {
        DIST_DIR = "dist"
        REMOTE_PATH = "/var/www/html/pulse-events"
        REMOTE_HOST = credentials('pulse-events-host')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Build static bundle') {
            steps {
                sh '''
                    rm -rf $DIST_DIR
                    mkdir -p $DIST_DIR
                    cp 2203149.html $DIST_DIR/index.html
                    cp events.html $DIST_DIR/events.html
                    cp 2203149.css $DIST_DIR/2203149.css
                '''
            }
        }

        stage('Archive artifact') {
            steps {
                dir("${DIST_DIR}") {
                    sh 'zip -r ../site.zip .'
                }
                archiveArtifacts artifacts: 'site.zip', fingerprint: true
            }
        }

        stage('Deploy to web server') {
            steps {
                sshagent (credentials: ['pulse-events-ssh']) {
                    sh '''
                        rsync -avz --delete $DIST_DIR/ ${REMOTE_HOST}:${REMOTE_PATH}/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }
        failure {
            echo 'Deployment failed. Check Jenkins logs.'
        }
    }
}

