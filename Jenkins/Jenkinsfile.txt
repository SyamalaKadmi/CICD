pipeline {
    agent any

    environment {
        VENV_PATH = "${WORKSPACE}"
    }

    triggers {
        pollSCM('H/5 * * * *')  // Checks for changes every 5 minutes
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SyamalaKadmi/FlaskTest.git'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'pytest --junitxml=test-results.xml'
                }
            }
        }

        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'nohup python3 app.py > flask.log 2>&1 &'
            }
        }
    }

    post {
        success {
            emailext subject: "Jenkins Build Success: ${JOB_NAME}",
                     body: "Build #${BUILD_NUMBER} for ${JOB_NAME} succeeded.\nCheck: ${BUILD_URL}",
                     to: 'syamala.kadimi@gmail.com'  // Ensure this is correct'
        }
        failure {
            emailext subject: "Jenkins Build Failed: ${JOB_NAME}",
                     body: "Build for ${JOB_NAME} failed.\nCheck logs: ${BUILD_URL}",
                     to: 'vinodmadaka_1993@outlook.com'
        }
    }
}
