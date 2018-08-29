pipeline {
    agent any

    environment {
        SLACK_CHANNEL = "#jenkins-deploy"
    }

    stages {
        stage('Build') {
            steps {
                //Builds the Artifacts and creates the docker image and tag
                sh "mvn clean package -Dbuild.number=${BUILD_NUMBER}"
            }

            post {
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Publish') {
            steps {

                //Deploys the artifacts to Archiva
                sh "mvn deploy -DskipTests"
                // this has to be deployed prior to running the maven stuff, since it tries to download the core library

                script {
                    env.APP = sh(returnStdout: true, script: "${JENKINS_HOME}/app.sh").trim()
                    env.VERSION = sh(returnStdout: true, script: "${JENKINS_HOME}/maven_version.sh").trim()
                }
            }
        }


    }
    post {
        success{
            slackSend channel:"${SLACK_CHANNEL}",
                    color: 'good',
                    message: "*Build Successful - ${env.JOB_NAME}* (<${env.BUILD_URL}|Open>)\nVersion - ${env.VERSION}"

        }
        failure{
            slackSend channel:"${SLACK_CHANNEL}",
                    color: 'danger',
                    message: "*Build Failure - ${env.JOB_NAME}* (<${env.BUILD_URL}|Open>)\nVersion - ${env.VERSION}"
        }
    }
}
