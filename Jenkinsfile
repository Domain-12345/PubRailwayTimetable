pipeline {
    agent any
    environment {
        //be sure to replace "ketanvj" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "smitasp/railwaytt"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sudo chmod +ux gradlew
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/railwaytt.zip'
            }
        }
        stage('Build Docker Image') {
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
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'railwaytt-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'railwaytt-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                
       /*           withKubeConfig([credentialsId: 'kubeconfig'
                    ]) {
                sh 'kubectl delete service railway-timetable-service-canary'
                sh 'kubectl delete deployment railway-timetable-deployment-canary'
    }*/
                
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'railwaytt-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
