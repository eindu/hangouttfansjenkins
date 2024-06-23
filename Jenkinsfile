pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID=credentials('AWS_ACCESS_KEY')
        AWS_SECRET_ACCESS_KEY=credentials('AWS_SECRET_ACCESS_KEY')
        ANSIBLE_HOST_KEY_CHECKING="FALSE"
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        timeout(time: 1, unit: 'HOURS')
    }
    tools {
        maven '3.8.7'
        terraform '40619'
    } 
    stages {
        stage('checkout') {
        steps {
            git(url: 'https://github.com/eindu/hangouttfansjenkins.git')
        }
    }
    stage('infra') {
        steps {
            sh '''
                terraform -chdir=src/main/config/terraform init
                terraform -chdir=src/main/config/terraform apply --auto-approve
                terraform -chdir=src/main/config/terraform output --raw "hangout_ec2ip" > hosts
            '''
        }
        post {
            failure {
                sh '''
                    terraform -chdir=src/main/config/terraform destroy --auto-approve
                '''
            }
        }
    }
    stage('build') {
      steps {
        sh 'mvn --batch-mode clean package'
      }
    }
    stage('deploy') {
        steps {
            ansiblePlaybook(playbook: 'src/main/config/ansible/hangout-playbook.yml', credentialsId: 'AWS_EC2_KEY', installation: '2.16.8', inventory: 'hosts')
        }
    }
  }
}
