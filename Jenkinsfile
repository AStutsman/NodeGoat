/*
 * Normal Jenkinsfile that will build and SCA scans
 */

pipeline {
    agent any

    environment {
        Sonatype_App_Name = 'NodeGoat'      // App Name in the Lifecycle Platform
        Sonatype_IQ_Server = 'https://astuts.ngrok.io'
    }

    // this is optional on Linux, if jenkins does not have access to your locally installed docker
    //tools {
        // these match up with 'Manage Jenkins -> Global Tool Config'
        //'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker-latest'
    //}

    options {
        // only keep the last x build logs and artifacts (for space saving)
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    }

    stages{
        stage ('Environment Verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        stage ('Build') {
            steps {
                // use the NodeJS plugin
                nodejs(nodeJSInstallationName: 'NodeJS-16.0.0') {
                    script {
                        if(isUnix() == true) {
                            //sh 'npm config ls'
                            sh 'npm --version'
                            sh 'npm install'
                        }
                        else {
                            bat 'npm --version'
                            bat 'npm install'
                        }
                    }

                }
            }
        }

        stage ('OSS Scan') {
            steps {
                withCredentials([ usernamePassword (
                    credentialsId: 'IQ_login', usernameVariable: 'IQ_User', passwordVariable: 'IQ_Key') ]) {
                sh '''
                echo "Beginning Sonatype OSS Scan"
                wget https://download.sonatype.com/clm/scanner/latest.jar -O /latest.jar
                java -jar /latest.jar -a "${IQ_User}":"${IQ_Key}" -D includeNpmDependencies -s $Sonatype_IQ_Server -i $Sonatype_App_Name -t stage-release ./
                '''
                }
            }
        }

        // only works on *nix, as we're building a Linux image
        //  uses the natively installed docker
        stage ('Deploy') {
            when { expression { return (isUnix() == true) } }
            steps {
                echo 'building Docker image'
                sh 'docker version'

                ansiColor('xterm') {
                    sh 'docker build -t nodegoat:${BUILD_TAG} .'
                }

                // split into separate stage??
                echo 'Deploying ...'

            }
        }
    }
}
