pipeline {
    agent { 
        label 'master'
    }
    environment {
        registry = 'dockeryuliya/sa_project'
        registryCredential = 'docker-hub-credentials'
    }
    stages {
        stage('Clear workspace') {
           steps {
              deleteDir()
            }
        }
        stage ("Clone repo") {
            steps {
                git url:'git@github.com:YuliyaHancharenka/sa_project.git',
                credentialsId: '66f6eb3b-3cd3-492c-997b-6579c2ab1049'
            }
        }
        stage ("Build image") {
            steps {
                sh "sudo docker build . -t ${registry}:${BUILD_NUMBER}"
            }
        }
        stage('Run container') {
          steps {
              sh """
                 docker run --rm -v \$(pwd):/ansible/playbooks \$registry:\$BUILD_NUMBER --version
                 """
            }
        }
        stage('Run ansible in prod') {
          steps {
              sh "ls -l"

              sh """
                 sudo docker run --rm \
                    -v ~/.ssh/id_rsa:/root/.ssh/id_rsa \
                    -v ~/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub \
                    -v \$(pwd):/ansible/playbooks \
                    \$registry:\$BUILD_NUMBER -i prod.yaml xwiki.yaml
                 """
            }
        }
        stage ("Push image to DockerHub") {
            steps {
                script {
                    "sudo " + docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage ("Remove image") {
            steps {
                sh "sudo docker rmi ${registry}:${BUILD_NUMBER}"
            }
        }
    }
    post{
        success{
            slackSend (color: '#00CC00', message: "Job ${env.JOB_NAME} finished successfully. Build: ${env.BUILD_NUMBER}")
        }
        failure{
            slackSend (color: '#FF0000', message: "Job ${env.JOB_NAME} failed. Build: ${env.BUILD_NUMBER}")
        }
    }
}
