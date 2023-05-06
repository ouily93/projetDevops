
pipeline{
    agent any
    options {
		
	}
    environment {
         APP_NAME = "aop"
        VERSION_IMAGE_APP = '0.0.1-SNAPSHOT'
         repo = "${JOB_NAME}"
         DOCKER_REGISTRY_USER= '128.8.9.1'
         DOCKER_REGISTRY_USER_PASSWORD= 'root'
         SONARQUBE_URL= '128.8.9.2'
         USER_PASSWORD_SONAR= 'root'
        }
    tools {
        maven 'maven'
        dockerTool 'DOCKER'
    }
    stages{
         stage("check Packages") {
            steps {
              sh script: 'mvn clean install -Dmaven.test.skip=true'
            }
        }

            stage('SCM Checkout') {
              steps {
                 git(branch: 'main', credentialsId: 'gitlab',  url:'127.0.0.1')                   
              }
         }
           stage('Check Packages'){
                steps{
                    sh script: 'mvn clean install -Dmaven.test.skip=true'
                }
            }

              stage('SonarQube analysis') {
                  steps{
                  withSonarQubeEnv(credentialsId: 'sonar', installationName:"Sonar") {
                      sh 'mvn sonar:sonar -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.login=${TOKEN_SONAR}'
                    }
                }
              }
        stage('Upload Jar to nexus'){
            steps{
                
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'aop',
                        classifier: '',
                        file: 'target/aop-0.0.1-SNAPSHOT.jar',
                        type: 'jar'
                    ]
                ],
                credentialsId: 'nexus',
                groupId: 'aop',
                nexusUrl: 'docker pull sonatype/nexus3',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'aop',
                version: '0.0.1-SNAPSHOT'
            }
        }
                    stage('Build Image'){
                          steps{
                            script {
                             sh "docker build -t ${DOCKER_REGISTRY_URL}/${env.imageName} ."
                            }
                          }
                        }
                    stage('Connect To Registry'){
                        steps{
                            sh "docker logout"
                            sh "docker login ${DOCKER_REGISTRY_URL} --username ${env.DOCKER_REGISTRY_USER} --password ${env.DOCKER_REGISTRY_USER_PASSWORD}"
                        }
                    }
                    stage ('Push Docker Image'){
                        steps{
                            script{
                          sh "docker push ${DOCKER_REGISTRY_URL}/${env.imageName}:latest" 
                         
                        }
                        }
                    }
                                        stage ('Delete Tempory Image'){
                        steps{
                        sh "docker rmi ${DOCKER_REGISTRY_URL}/${env.imageName}:latest"
                    }
                    }


             stage('Deploy to cluster') {
                  steps {
                    script {
                       sshagent(['aopdeploy']) {
                           sh'ssh -oStrictHostKeyChecking=no root@10.0.0.20 kubectl delete deploy --ignore-not-found=true aop-app'
                                                      sh'ssh -oStrictHostKeyChecking=no root@10.0.0.20 kubectl delete svc --ignore-not-found=true aop-svc'
                                     sh 'ssh -oStrictHostKeyChecking=no root@10.0.0.20 kubectl apply -f /var/lib/jenkins/workspace/aop_main@2/deploymentservice.yml'
                                     sh 'sleep 30'
                                }
                        
                    }
                  }
                }

                        }
                            	post {
   			 failure {
       			 mail to: 'ouily@gmail.com ',
             		subject: "**Failed Pipeline**: ${currentBuild.fullDisplayName}",
             		body: "Something is wrong with ${env.BUILD_URL}"
    }
             success{
                mail to: 'ouily@gmail.com ',
                 subject: "**Success Pipeline**:${currentBuild.fullDisplayName}",
           		    body: "Success of your build, here is the link of the build ${env.BUILD_URL}"
                        }
}
              }
