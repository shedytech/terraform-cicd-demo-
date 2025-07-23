pipeline {
    agent any
    environment {
        TERRAFORM_VERSION = '1.5.7'
        TF_WORKING_DIR = 'terraform'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Terraform Init - Dev') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform init"
                }
            }
        }
        stage('Terraform Plan - Dev') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform plan -var-file=dev.tfvars -out=tfplan"
                }
            }
        }
        stage('Terraform Apply - Dev') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform apply -auto-approve tfplan"
                }
            }
        }
        stage('Test - Dev') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terratest test-dev-infra.go"
                }
            }
        }
        stage('Peer Review - Staging') {
            steps {
                input message: 'Approve deployment to Staging?', ok: 'Deploy'
            }
        }
        stage('Terraform Plan - Staging') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform plan -var-file=staging.tfvars -out=tfplan"
                }
            }
        }
        stage('Terraform Apply - Staging') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform apply -auto-approve tfplan"
                }
            }
        }
        stage('Test - Staging') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terratest test-staging-infra.go"
                }
            }
        }
        stage('Peer Review - Production') {
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
            }
        }
        stage('Terraform Plan - Production') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform plan -var-file=prod.tfvars -out=tfplan"
                }
            }
        }
        stage('Terraform Apply - Production') {
            steps {
                dir(TF_WORKING_DIR) {
                    sh "terraform apply -auto-approve tfplan"
                }
            }
        }
        stage('Monitoring Setup') {
            steps {
                sh "curl -X POST http://prometheus:9090/api/v1/alerts -d @alerts.json"
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "${TF_WORKING_DIR}/tfplan", allowEmptyArchive: true
            cleanWs()
        }
        success {
            slackSend channel: '#deployments', message: "✅ Terraform deployment succeeded for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        failure {
            slackSend channel: '#deployments', message: "❌ Terraform deployment failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
