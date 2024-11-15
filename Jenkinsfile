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
        stage('Security Scan') {
            steps {
                sh '''
                    oc process -f kubefiles/security-scan-template.yml \
                    -n eyrptl-monitoring-lab \
                    -p QUAY_USER=marchioni_francesco \
                    -p QUAY_REPOSITORY=do400-monitoring-lab \
                    -p APP_NAME=calculator \
                    -p CVE_CODE=CVE-2021-23840 \
                    | oc replace --force \
                    -n eyrptl-monitoring-lab -f -
                '''
                sh '''
                    ./scripts/check-job-state.sh "calculator-trivy" \
                    "eyrptl-monitoring-lab"
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    ./scripts/redeploy.sh \
                        -d calculator \
                        -n eyrptl-monitoring-lab
                '''
            }
        }
    }
    post {
        failure {
            mail to: "team@example.com",
                subject: "Pipeline failed: ${currentBuild.fullDisplayName}",
                body: "The following pipeline failed: ${env.BUILD_URL}"
        }
    }
}
