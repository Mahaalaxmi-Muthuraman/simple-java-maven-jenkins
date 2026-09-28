pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Secret Scan - Gitleaks') {
            steps {
                sh 'gitleaks detect --source . --verbose'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'nvd-api-key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    dependencyCheck(
                        additionalArguments: "--scan . --format XML --format HTML --nvdApiKey ${NVD_API_KEY}",
                        odcInstallation: 'OWASP-Dependency-Check'
                    )
                }
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                    fingerprint: true
            }
        }
    }

    post {
        always {
            dependencyCheckPublisher(
                pattern: '**/dependency-check-report.xml'
            )
        }
    }
}
