pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'Maven_Latest'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RiddhiPatil15/SeleniumJavaCapstoneProject.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                bat 'mvn clean'
            }
        }

        stage('Run UI + API + E2E + Negative Tests') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Clean JMeter Results') {
            steps {
                bat '''
                if exist jmeter\\results\\results.jtl del /f /q jmeter\\results\\results.jtl
                if exist jmeter\\reports rmdir /s /q jmeter\\reports
                if not exist jmeter\\results mkdir jmeter\\results
                if not exist jmeter\\reports mkdir jmeter\\reports
                '''
            }
        }

        stage('Run JMeter Performance Tests') {
            steps {
                bat '''
                jmeter -n -t jmeter/testplan.jmx ^
                -l jmeter/results/results.jtl ^
                -e -o jmeter/reports
                '''
            }
        }

        stage('Generate Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }

        stage('Publish HTML Report') {
            steps {
                publishHTML([
                    reportDir: 'allure-report',
                    reportFiles: 'index.html',
                    reportName: 'Allure Report',
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'target/surefire-reports/**', fingerprint: true
            archiveArtifacts artifacts: 'allure-report/**', fingerprint: true
            archiveArtifacts artifacts: 'jmeter/reports/**', fingerprint: true
            archiveArtifacts artifacts: 'jmeter/results/**', fingerprint: true
        }

        success {
            echo 'Pipeline executed successfully'
        }

        failure {
            echo 'Pipeline failed - check logs'
        }
    }
}
