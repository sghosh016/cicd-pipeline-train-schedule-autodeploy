pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "lusifer09/train-schedule"
    }
    tools {
        jdk 'java'
    }
    stages {
        stage('Build Project') {
            steps {
                echo 'Running build automation'
           //     sh './gradlew build --no-daemon'
             //   archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Docker Image Creation') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image to Hub') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            agent {
                label 'kube-master'
            }
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
               // kubernetesDeploy(
                 //   kubeconfigId: 'kubeconfig',
                  //  configs: 'train-schedule-kube-canary.yml',
                   // enableConfigSubstitution: true
                //) 
                sh 'kubectl apply -f train-schedule-kube-canary.yml --kubeconfig=/home/ubuntu/.kube/config'
            }
        }
        stage('DeployToKubernetes') {
            agent {
                label 'kube-master'
            }
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
       //         kubernetesDeploy(
         //           kubeconfigId: 'kubeconfig',
           //         configs: 'train-schedule-kube-canary.yml',
             //       enableConfigSubstitution: true
               // )
                sh 'kubectl apply -f train-schedule-kube-canary.yml --kubeconfig=/home/ubuntu/.kube/config'
            }
        }
    }
}
