pipeline {
    agent any

    environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
    
    stages {

        stage('SCM checkout') {
            steps {
                echo 'checkout starts'
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/EbYVarghese18/appnginx-build-to-deployment']])
                echo 'checkout ends'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Build Dockerimage starts'
                script{
                    sh 'docker build -t myapp-nginx:${TAG} .'
                }
                echo 'Build Dockerimage ends'
            }
        }
        
        stage('Login to AWS ECR') {

			steps {
                echo 'Login to AWS ECR starts'
                script{
                    withCredentials([[
                    $class: 'AmazonWebServiceCredentialsBinding',
                    credentialsId: 'aws-ecr-access-key',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){

                        sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/j9i5q7x1'

                    }
                }   
			}
		}
        
        stage('Tag and Push image to AWS ECR') {
            
			steps {
                sh 'docker tag myapp-nginx:${TAG} public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
				sh 'docker push public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
			}
		}
	}
}   