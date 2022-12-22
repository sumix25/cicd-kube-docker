Skip to content
Search or jump to…
Pull requests
Issues
Codespaces
Marketplace
Explore

@sumix25
devopshydclub
vprofile-project
Public
Code
Issues
10
Pull requests
38
Actions
Projects
Security
Insights
vprofile-project/Jenkinsfile
@imran78688
imran78688 paac update 4
Latest commit 9f4dbfb on Oct 13, 2020
 History
 2 contributors
@imran78688@devopshydclub
127 lines (111 sloc)  4.17 KB

pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "kubesum/vprofileapp"
        registryCredential = "dockerhub"

    }

    stages{


        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image') {
           steps {
             script {
               dockerImage = docker.build registry + ":V$BUILD_NUMBER"
             }
           }
        }

        stage ('Upload Image'){
           steps {
              script {
                 docker.withRegistry('',registryCredential) {
                     dockerImage.push("V$BUILD_NUMBER")
                     dockerImage.push('latest')
                 }
              }
           }
        }

        stage (' Remove Unused docker image') {
           steps {
              sh "docker rmi $registry:V$BUILD_NUMBER"
           }
        }

        stage ('Kubernetes Deploy') {
          agent {label 'KOPS'}
           steps {
             sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
           }
        }

    }



}

Footer
© 2022 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
vprofile-project/Jenkinsfile at paac · devopshydclub/vprofile-project