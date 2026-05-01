pipeline {
    agent any

    environment {
        REPO_NAME    = "ramesh040183"
        IMAGE_NAME   = "foodweb1"
        IMAGE_TAG    = "latest"
        AWS_REGION   = "ap-south-1"
        CLUSTER_NAME = "my-cluster"
    }

    stages {

        stage('Git_Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/ramesh040183-code/AWS-JENKINS-EKS-Deployment.git'
            }
        }

        stage('Infra_Apply') {
            steps {
                dir('terraform') {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'AWS_Cred',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {

                        bat '''
                            terraform init
                            terraform validate
                            terraform plan
                            terraform apply -auto-approve
                        '''
                    }
                }
            }
        }

        stage('Docker_Build_Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Dockerhub-creds',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {

                    bat '''
                        docker build -t %USERNAME%/%IMAGE_NAME%:%IMAGE_TAG% .

                        echo %PASSWORD% | docker login -u %USERNAME% --password-stdin

                        docker push %USERNAME%/%IMAGE_NAME%:%IMAGE_TAG%
                    '''
                }
            }
        }

        stage('EKS_Deployment') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS_Cred',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {

                    bat '''
                        aws eks update-kubeconfig --name %CLUSTER_NAME% --region %AWS_REGION%

                        kubectl get nodes

                        kubectl apply -f K8S/deployment.yaml
                        kubectl apply -f K8S/service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!!'
        }

        failure {
            echo 'Pipeline failed!! Check logs!!'
        }
    }
}