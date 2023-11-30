pipeline {
    agent any
    tools{
        maven 'maven_3_5_0'
    }
    stages{
        stage('Build Maven'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Java-Techie-jt/devops-automation']]])
                sh 'mvn clean install'
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t mahadishohag/devops-integration .'
                }
            }
        }
        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                   sh 'docker login -u mahadishohag -p ${dockerhubpwd}'

}
                   sh 'docker push mahadishohag/devops-integration'
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    kubernetesDeploy (configs: 'deploymentservice.yaml',kubeconfigId: 'k8sconfigpwd')
                }
            }
        }
    }
}

// from shohag
pipeline {
    agent any
    stages {
        stage('git initial') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mhs-mahadi/devops-automation']])
            }
        }
        stage('Build Maven') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Build docker images') {
            steps {
                script {
                    def imageName = "mahadishohag/devops-integration:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                    // sh "docker tag ${imageName} ${imageName}-latest"
                }
            }
        }
        stage('Login') {
            steps {
                script {
                    def registry = 'https://index.docker.io/v1/'
                    def credentialsId = 'dockerhub'

                    withCredentials([usernamePassword(credentialsId: credentialsId, usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin $registry"
                    }
                }
            }
        }
        stage('Push image to Hub') {
            steps {
                script {
                    def imageName = "mahadishohag/devops-integration:${env.BUILD_NUMBER}"
                    withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                        docker.image(imageName).push()
                    }
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
    }
}
