pipeline {
    agent any
    stages {
        stage ('Build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker build -t ${user}/infra:${currentBuild.number} ."
                    sh "docker tag ${user}/infra:${currentBuild.number} ${user}/infra:latest"
                }
            }
        }
        stage ('Push to registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh "docker login -u ${user} -p ${pass}"
                    sh "docker push ${user}/infra:${currentBuild.number}"
                    sh "docker push ${user}/infra:latest"
                }
            }
        }
        stage ('Deploy on EKS') {
             steps {
                 dir('terraform') {
                   withAWS(region: 'ap-southeast-1', credentials: 'aws') {
                    sh "terraform init -reconfigure"
                    sh "terraform plan"
                    sh "terraform apply --auto-approve"
                     }
                    }
             }
        }
        stage ("Deploy pods on cluster") {
            steps {
                 withAWS(region: 'ap-southeast-1', credentials: 'aws') {
                      sh "aws eks update-kubeconfig --region ap-southeast-1 --name tss-cluster"
                      sh "kubectl apply -f my-deployment.yaml"
                 }
            }
        }
    }
}
