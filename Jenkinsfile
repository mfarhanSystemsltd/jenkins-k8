pipeline {
    agent {
	    label 'kubeagent'
    }
    environment {
       awstoken = ''
    }
    stages {
        stage('Git clone'){
            steps{
                git branch: 'main', url: 'https://github.com/mfarhanSystemsltd/ACE-HelloWorld.git'
            }
        }
        stage('Build docker image'){
            steps{
                //Build for AWS ECR
                sh 'docker build -t 736505676141.dkr.ecr.us-east-2.amazonaws.com/ace-test:${BUILD_NUMBER} .'
                //Build for Local repo
                //sh 'docker build -t 20.54.72.51:443/ace-test:${BUILD_NUMBER} .'
            }
        }
        stage('Push to AWS ECR'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'aws-ecr', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 736505676141.dkr.ecr.us-east-2.amazonaws.com'
                    sh 'docker push 736505676141.dkr.ecr.us-east-2.amazonaws.com/ace-test:${BUILD_NUMBER}'
                }
            }
        }
        // stage('Push to Docker local registry'){
        //     steps{
        //          sh 'docker push 20.54.72.51:443/ace-test:${BUILD_NUMBER}'
        //     }
        // }
        stage('Pull Helm chart'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'aws-ecr', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh 'helm registry login --username AWS --password $(aws ecr get-login-password --region us-east-2) 736505676141.dkr.ecr.us-east-2.amazonaws.com'
                    sh 'export HELM_EXPERIMENTAL_OCI=1'
                    sh 'helm pull oci://736505676141.dkr.ecr.us-east-2.amazonaws.com/ace-deployment --version 0.1.0'
                    sh 'tar -zxvf *.tgz'
                }
            }
        }
        stage('Installing/Upgrading the Helm chart using ECR image'){
            steps{
                kubeconfig(credentialsId: 'mykubeconfig', serverUrl: '', caCertificate: '') {
                        sh 'helm upgrade ace-chart ace-deployment/ --set replicaCount=3 --set image.name=736505676141.dkr.ecr.us-east-2.amazonaws.com/ace-test:${BUILD_NUMBER}'
                        sh 'helm list'
                        //sh 'helm rollback ace-chart'
                    
                }  
            }
        }
        // stage('Installing/Upgrading the Helm chart using local docker registry image'){
        //     steps{
        //         kubeconfig(credentialsId: 'mykubeconfig', serverUrl: '', caCertificate: '') {
        //             sh 'helm upgrade ace-chart ace-deployment/ --set replicaCount=3 --set image.name=20.54.72.51:443/ace-test:${BUILD_NUMBER}'
        //         }  
        //     }
        // }
        // stage('Deploy to K8s') {
        //     steps {
        // 		kubeconfig(credentialsId: 'mykubeconfig', serverUrl: '', caCertificate: '') {
        // 		    sh 'kubectl apply -f deployment.yaml'
        // 		}
        //     }
        // }
    }
}
