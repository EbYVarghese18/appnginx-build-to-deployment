pipeline {
    agent any

    environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
        AWS_OUTPUT_FORMAT = 'json'
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-access-key', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                    sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                    sh 'aws configure set default.region $AWS_REGION'
                    sh 'aws configure set default.output $AWS_OUTPUT_FORMAT'
                }
                script{
                    sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/j9i5q7x1'
                }
                echo 'Login to AWS ECR ends'
            }
		}
        
        stage('Tag and Push image to AWS ECR') {
			steps {
                script{
                    echo 'Push the image to ECR starts'
                    sh 'docker tag myapp-nginx:${TAG} public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
                    sh 'docker push public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
                    echo 'Push the image to ECR ends'
                }
			}
		}

        stage('Helm Push') {
            steps {
                script{
                    sh 'helm create myapp-nginx-helm'
                    // sh 'pwd'
                    // sh 'ls -al'
                    sh 'echo version : 0.${BUILD_NUMBER}.0 >> myapp-nginx-helm/Chart.yaml'
                    sh "sed -i 's|tag: \".*\"|tag: \"${BUILD_NUMBER}\"|' myapp-nginx-helm/values.yaml"
                    sh 'helm package myapp-nginx-helm'
                    // sh 'aws ecr get-login-password  --region us-east-1 | helm registry login --username AWS --password-stdin 976846671615.dkr.ecr.us-east-1.amazonaws.com'
                    // sh 'helm push myapp-nginx-helm-0.${BUILD_NUMBER}.0.tgz oci://976846671615.dkr.ecr.us-east-1.amazonaws.com'
                    // sh 'rm -rf myapp-nginx-helm-*'
                }
            }
        }

        // stage('Invoke Build number to Pipeline B') {
        //     steps {
        //         build job: 'sample-maven-project-docker-deployment', parameters : [[ $class: 'StringParameterValue', name: 'buildnum', value: "${BUILD_NUMBER}"]]
        //     }
        // }

	}
}