pipeline {
    agent any
    tools {
        // Install the Maven version configured as "maven-3.6.2" and add it to the path.
        maven "Maven-3.6.2"
        jdk "JDK8"
    }

    stages {
        stage('Checkout') {
            steps {
                // Get some code from a GitHub repository
                checkout scm
            }
        }
        stage('Build') {
            steps {
                dir("app"){
                    withMaven{
 
                        sh 'mvn package'
                        //sh "mvn install"
                        //sh "mvn org.codehaus.mojo:versions-maven-plugin:2.8.1:display-version -q -DforceStdout -m 'Tagging release `mvn org.codehaus.mojo:versions-maven-plugin:2.8.1:display-version -q -DforceStdout`"
                        // sh "git tag -a 'successful build' && git push --tags"

                    }

                }
            }
        }
        stage('Add tag'){
            steps{
                dir("app"){
                    script{
                        def pomXml = readMavenPom file: 'pom.xml'
                        echo "pomXml: $pomXml"
                        nextVersion = pomXml.version
                        echo "${nextVersion}"
                        // sh "git tag -a '${nextVersion}' -m '${nextVersion}'" 
                                sh '''
                        TAG_NAME="${nextVersion}"
                        if git tag -l $TAG_NAME > /dev/null 2>&1; then
                            echo "Tag $TAG_NAME already exists"
                        else
                            echo "Creating tag $TAG_NAME"
                            git tag $TAG_NAME
                        fi
                        '''
                    }
                }
            }
        }
        stage('test'){
            steps{
                
                dir("app"){
                    sh "mvn test"
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('archive'){
            steps{
                dir("app"){
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
    post {
        always {
            script {
                def currentBuildStatus = currentBuild.result

                if (currentBuildStatus == 'SUCCESS') {
                    slackSend(
                        color: "#00FF00",
                        channel: "jenkins-notify",
                        message: "${currentBuild.fullDisplayName} succeeded",
                        tokenCredentialId: 'slack-token'
                    )
                } else {
                    slackSend(
                        color: "#FF0000",
                        channel: "jenkins-notify",
                        message: "${currentBuild.fullDisplayName} was failed",
                        tokenCredentialId: 'slack-token'
                    )
                }
            }
        }
    }      
}