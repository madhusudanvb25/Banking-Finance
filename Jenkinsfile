pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        ECR_REPO = '491085389171.dkr.ecr.ap-south-1.amazonaws.com/financeme' // From Terraform output
        RDS_ENDPOINT = 'terraform-20250515035808999200000001.c92aiw0gsl41.ap-south-1.rds.amazonaws.com:3306' // From Terraform output
    }
    stages {
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://github.com/madhusudanvb25/Banking-Finance.git'
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t financeme:${BUILD_NUMBER} .'
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REPO}'
                    sh 'docker tag financeme:${BUILD_NUMBER} ${ECR_REPO}:${BUILD_NUMBER}'
                    sh 'docker push ${ECR_REPO}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Test') {
            steps {
                ansiblePlaybook(
                    playbook: 'deploy_to_test.yml',
                    inventory: 'inventory.ini',
                    credentialsId: 'ansible-ssh-key',
                    extraVars: [
                        docker_image: "${ECR_REPO}:${BUILD_NUMBER}",
                        db_url: "jdbc:mysql://${RDS_ENDPOINT}/financeme",
                        db_username: "admin",
                        db_password: "admin123456"
                    ]
                )
            }
        }
        stage('Run Selenium Tests') {
            steps {
                // Placeholder: Implement Selenium tests
                echo 'Running Selenium tests on test.themadhu.shop'
            }
        }
        stage('Deploy to Prod') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                ansiblePlaybook(
                    playbook: 'deploy_to_prod.yml',
                    inventory: 'inventory.ini',
                    credentialsId: 'ansible-ssh-key',
                    extraVars: [
                        docker_image: "${ECR_REPO}:${BUILD_NUMBER}",
                        db_url: "jdbc:mysql://${RDS_ENDPOINT}/financeme",
                        db_username: "admin",
                        db_password: "admin123456"
                    ]
                )
            }
        }
    }
}
