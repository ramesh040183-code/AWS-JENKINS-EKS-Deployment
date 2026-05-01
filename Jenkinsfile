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

                        aws eks update-kubeconfig --name %CLUSTER_NAME% --region %AWS_REGION%

                        kubectl get nodes

                        kubectl delete -f K8S/deployment.yaml
                        kubectl delete -f K8S/service.yaml

                        terraform destroy -auto-approve
                            
                        '''
                    }
                }
            }
        }  
    }
}