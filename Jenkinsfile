pipeline {
    agent any
    environment {
        PROJECT_ID = 'ozanoguzgkeproject'
        CLUSTER_NAME = 'mycluster'
        LOCATION = 'europe-west3-a'
        CREDENTIALS_ID = 'ozanoguzgkeproject'
    }
    stages {
        stage('Checkout SCM') {
            steps {
                checkout([
                 $class: 'GitSCM',
                 branches: [[name: 'master']],
                 userRemoteConfigs: [[
                    url: 'git@github.com:ozanoguz/gkecwp.git',
                    credentialsId: '',
                 ]]
                ])
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
                    fortiCWPScanner block: true, imageName: "ozanoguz/hello:${env.BUILD_ID}" 
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