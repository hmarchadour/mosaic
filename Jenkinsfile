pipeline {
    agent 'any'

    tools {
        maven 'apache-maven-3.6.3'
        jdk 'adoptopenjdk-hotspot-jdk8-latest'
    }

    stages {
        stage('Build') {
            steps {
                withMaven(mavenLocalRepo: '.repository', publisherStrategy: 'EXPLICIT') {
                    sh 'mvn clean install -DskipTests -fae -T 4'
                }
            }
        }

        stage('Test') {
            steps {
                withMaven(mavenLocalRepo: '.repository', publisherStrategy: 'EXPLICIT') {
                    sh 'mvn test -fae -T 4'
                }
            }

            post {
                always {
                    junit '**/surefire-reports/*.xml'
                }
            }
        }

//        stage('Integration Tests') {
//            steps {
//                withMaven(mavenLocalRepo: '.repository', publisherStrategy: 'EXPLICIT') {
//                    sh 'mvn test -fae -P integration-tests'
//                }
//            }
//
//            post {
//                always {
//                    junit 'test/**/surefire-reports/*.xml'
//                }
//            }
//        }

        stage('Analysis') {
            steps {
                withMaven(mavenLocalRepo: '.repository', publisherStrategy: 'EXPLICIT') {
                    sh 'mvn site -T 4'
                }
            }

            post {
                always {
                    recordIssues(sourceCodeEncoding: 'UTF-8', tools: [
                            spotBugs(),
                            checkStyle(),
                            taskScanner(highTags: 'FIXME', normalTags: 'TODO', ignoreCase: true, includePattern: '**/*.java')
                    ])
                }
            }
        }

        stage('Deploy') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                withMaven(mavenLocalRepo: '.repository', publisherStrategy: 'EXPLICIT') {
                    sh 'mvn deploy'
                }
            }
        }
    }

    post {
        success {
            cleanWs()
        }
        fixed {
            sendMail("Successful")
        }
        unstable {
            sendMail("Unstable")
        }
        failure {
            sendMail("Failed")
        }
    }
}

def sendMail(status) {
    emailext(
            recipientProviders: [culprits()],
            subject: status + ": ${currentBuild.fullDisplayName}",
            body: '${JELLY_SCRIPT, template="html-with-health-and-console"}',
            mimeType: 'text/html'
    )
}