pipeline {
    agent { node { label 'maven' } }
    stages {
        stage('Test') {
            steps {
                sh './mvnw clean test'
            }
        }
        stage('Build') {
            environment {
                QUAY = credentials('QUAY_USER')
            }
            steps {
                sh './scripts/include-container-extensions.sh'

                sh '''
                    ./scripts/build-and-push-image.sh \
                    -b $BUILD_NUMBER \
                    -u "$QUAY_USR" \
                    -p "$QUAY_PSW" \
                    -r do400-monitoring-lab
                '''
            }
        } 
    }
}