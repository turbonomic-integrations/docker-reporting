pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'ls -alh'
                sh 'docker build -f Dockerfile.alpine -t turbointegrations/reporting:0.1-alpine .'
                sh 'docker build -f Dockerfile.slim-buster -t turbointegrations/reporting:0.1-slim-buster .'
                sh 'docker build -f Dockerfile.rhel -t turbointegrations/reporting:0.1-rhel .'
            }
        }

        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DHPASS', usernameVariable: 'DHUSER')]) {
                    sh 'docker login -u $DHUSER -p $DHPASS'
                    sh 'docker push turbointegrations/reporting:0.1-alpine'
                    sh 'docker push turbointegrations/reporting:0.1-slim-buster'
                    sh 'docker push turbointegrations/reporting:0.1-rhel'
                }
            }
        }
    }
}
