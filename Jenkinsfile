pipeline {
    agent any

    environment {
        // DATE = new Date().format('yy.M')
        // TAG = "${DATE}.${BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
        AWS_OUTPUT_FORMAT = 'json'
        CHART_NAME = 'myapp-nginx-helm'
        CHART_VERSION = '1.0.0'
        ECR_REPOSITORY = 'public.ecr.aws/j9i5q7x1/myapp-nginx'
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
                    sh 'docker build -t myapp-nginx:${BUILD_NUMBER} .'
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
                    sh 'docker tag myapp-nginx:${BUILD_NUMBER} public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
                    sh 'docker push public.ecr.aws/j9i5q7x1/myapp-nginx:latest'
                    echo 'Push the image to ECR ends'
                }
			}
		}

        stage('Build Helm chart and Push to ECR') {
            steps {
                script{
                    echo 'creating helm chart'
                    sh "helm create ${CHART_NAME}"

                    echo 'updating image tag in value file'
                    sh "sed -i 's|tag: \".*\"|tag: \"${BUILD_NUMBER}\"|' ${CHART_NAME}/values.yaml"

                    echo 'Builing helm package'
                    sh "helm package ${CHART_NAME} --version ${CHART_VERSION}"
                    // sh "helm package ${CHART_NAME} --version ${CHART_VERSION}"

                    // sh 'zip -r myapp-nginx-helm.zip myapp-nginx-helm'

                    echo 'pushing the package zip file to ECR'
                    sh "helm chart save ${CHART_NAME}-${CHART_VERSION}.tgz ${ECR_REPOSITORY}/${CHART_NAME}:${CHART_VERSION}"
                    sh "helm chart push ${ECR_REPOSITORY}/${CHART_NAME}:${CHART_VERSION}"

                    // sh 'helm push '

                    // echo 'Cleanig up the files'
                    // sh 'rm -rf myapp-nginx-helm'
                    // sh 'rm -rf myapp-nginx-helm.zip'
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