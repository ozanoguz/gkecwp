pipeline {
    agent any
    environment {
        PROJECT_ID = 'forti-emea-se'
        CLUSTER_NAME = 'oogkecluster'
        LOCATION = 'europe-west3-a'
        CREDENTIALS_ID = 'forti-emea-se'
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
                
            }
        }
        stage("FortiDevSec SAST") {
            steps {
sh 'docker pull registry.fortidevsec.forticloud.com/fdevsec_sast:latest'
   sh 'docker run –rm –mount type=bind,source=$pwd,target=/scan registry.fortidevsec.forticloud.com/fdevsec_sast:latest'
              }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("ozanoguz/hello:${env.BUILD_ID}")
                }
            }
        }
        stage("FortiCWP Image Scan") {
            steps {
                script {
                     try {
                        fortiCWPScanner block: true, imageName: "ozanoguz/hello:${env.BUILD_ID}"
                        } catch (Exception e) {
    
                 echo "Request for Approval"  
                  }
                }
            }
        }
             stage('Code approval request') {
     
           steps {
             script {
               def userInput = input(id: 'confirm', message: 'Do you Approve to use this code?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Approve Code to Proceed', name: 'approve'] ])
              }
            }
          }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }    
}