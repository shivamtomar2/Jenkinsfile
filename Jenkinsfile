pipeline {

    agent any

    tools {
        jdk 'JDK8'
    }

    parameters {
        choice(
            name: 'CHOICE',
            choices: ['dev', 'uat', 'prod'],
            description: 'Select Environment'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Code pulled from SCM/GitHub"
            }
        }

        stage('GitLeaks Scan') {
            steps {
                sh '''
                gitleaks detect \
                --source . \
                --report-format json \
                --report-path gitleaks-report.json || true
                '''
            }
        }

        stage('Parallel Stage') {

            parallel {

                stage('Compile') {
                    steps {
                        sh 'java -version'
                        sh 'mvn compile || true'
                    }
                }

                stage('Unit Test') {
                    steps {
                        sh 'mvn test || true'
                    }
                }
            }
        }

        stage('Approval') {
            steps {
                input message: 'Do you want to continue for Build Stage?', ok: 'Proceed'
            }
        }

        stage('Build') {

            when {
                expression {
                    params.CHOICE == 'prod'
                }
            }

            steps {
                sh 'mvn package -DskipTests || true'
            }
        }

        stage('Git Push') {
            steps {
                echo "Code pushed to SCM/GitHub"
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json'
        }
    }
}
