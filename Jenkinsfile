pipeline {
    agent any 

    environment {
        REPO_SERVER = "339712792713.dkr.ecr.us-east-1.amazonaws.com"
        REPO_NAME_BACKEND = "${REPO_SERVER}/badreads-backend"
        REPO_NAME_FRONTEND = "${REPO_SERVER}/badreads-frontend"
        IMAGE_VERSION = "${BUILD_NUMBER}"
    }

    stages {

        stage("build image") {
            steps {
                script {
                    echo "building docker images ..."
                    withCredentials([
                        usernamePassword(credentialsId: 'ecr-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')
                    ]){
                        sh "docker login -u ${USER} -p ${PASS} ${REPO_SERVER}"
                        sh "docker build badreads-backend/. -t ${REPO_NAME_BACKEND}:${IMAGE_VERSION}"
                        sh "docker push ${REPO_NAME_BACKEND}:${IMAGE_VERSION}"
                        sh "docker build badreads-frontend/. -t ${REPO_NAME_FRONTEND}:${IMAGE_VERSION}"
                        sh "docker push ${REPO_NAME_FRONTEND}:${IMAGE_VERSION}"
                    }
                }
            }
        }

        stage("add kubeconfig") {
            when{
                expression {
                    BUILD_NUMBER == "1"
                }
            }
            environment {
                AWS_ACCESS_KEY_ID = credentials("aws_access_key_id")
                AWS_SECRET_ACCESS_KEY = credentials("aws_secret_access_key")
            }
            steps {
                echo 'Deploying to eks cluster ...'
                withCredentials([file(credentialsId:'kube-config', variable:'KUBECONFIG')]){
                script{
                    sh 'kubectl --kubeconfig=KUBECONFIG apply -f k8s'
                    }
                }
            }
        }

        stage("change image version") {
            steps {
                script {
                    echo "change image version .."
                    sh "sed -i \"s|image:.*|image: ${REPO_NAME_BACKEND}:${IMAGE_VERSION}|g\" k8s/backend.yaml"
                    sh "sed -i \"s|image:.*|image: ${REPO_NAME_FRONTEND}:${IMAGE_VERSION}|g\" k8s/frontend.yaml"
                }
            }
        }

        stage('Deploy to minikube') {
            environment {
                AWS_ACCESS_KEY_ID = credentials("aws_access_key_id")
                AWS_SECRET_ACCESS_KEY = credentials("aws_secret_access_key")
            }
            steps {
                echo 'Deploying to eks cluster ... '
                withCredentials([file(credentialsId:'kube-config', variable:'KUBECONFIG')]){
                script{
                    sh 'kubectl apply -f k8s'
                    }
                }
            }
        }
    }
}