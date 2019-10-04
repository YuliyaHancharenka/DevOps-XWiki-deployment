pipeline {
    agent { label 'master' }
    triggers { pollSCM('* * * * *') }
    environment {
      registry = "dockeryuliya/ansible"
  }
     
    stages {
        stage('Clear workspace') {
           steps {
              deleteDir()
            }
        }
        
        stage('Clone repository') { 
            steps { 
                git branch: "master", url: "git@github.com:YuliyaHancharenka/sa_project.git"
            }
        }

        stage('Building image') {
            steps {
              script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
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
                 docker run --rm \
                    -v ~/.ssh/id_rsa:/root/.ssh/id_rsa \
                    -v ~/.ssh/id_rsa.pub:/root/.ssh/id_rsa.pub \
                    -v \$(pwd):/ansible/playbooks \
                    \$registry:\$BUILD_NUMBER -i prod.yaml xwiki.yaml
                 """
            }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
    }


    post {
            success {
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
            failure {
                slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
}
