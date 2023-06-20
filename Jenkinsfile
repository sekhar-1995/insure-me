pipeline {
    agent any 
    stages {
        stage("Code Checkout") {
            steps {
                git changelog: false, credentialsId: 'ForGitHub', poll: false, url: 'https://github.com/sekhar-1995/insure-me.git'
            }
        }
        
        stage("Code Build") {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage("Image Prune") {
            steps {
                sh 'sudo docker image prune -a -f'
            }
        }
        
        stage("Publish HTML Reports") {
            steps {
               publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/etc/insure-me/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
        
        stage("Docker Image") {
            steps {
                sh 'sudo docker build -t insure-me .'
            }
        }
        
       stage("Push Docker Image To DockerHub") {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'DockerHubCRED', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DockerHubPassword')
                ]) {
                    script {
                        sh "echo ${DockerHubPassword} | sudo docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "sudo docker tag insure-me subham742/insure-me:latest"
                        sh "sudo docker push subham742/insure-me:latest"
                    }
                }
            }
        }
        
        stage("Configure and Deploy to the test-server") {
    steps {
        ansiblePlaybook credentialsId: 'SSH-for-ansible', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/hosts', playbook: '/etc/ansible/playbook.yml'
           }
        }
    }
}
