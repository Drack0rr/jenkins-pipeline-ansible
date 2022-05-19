/* import shared library */
@Library('shared-library')_


pipeline {
    agent none
    stages {
        stage('Check yaml syntax') {
            agent { docker { image 'sdesbure/yamllint' } }
            steps {
                sh 'yamllint --version'
                sh 'yamllint \${WORKSPACE}'
            }
        }
        stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
                sh 'apk --no-cache add git'
                sh 'gem install mdl'
                sh 'mdl --version'
                sh 'mdl --style all --warnings --git-recurse \${WORKSPACE}'
            }
        }
        stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
            }
        }
        stage('Test and deploy the application') {
            environment {
                SUDOPASS = credentials('sudopass')
		USER = credentials('user-ansible')
            }
            agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
            stages {
               stage("Verify ansible playbook syntax deploy") {
                   steps {
                       sh 'ansible-lint deploy.yml'
                   }
               }
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       apt-get update
                       apt-get install -y sshpass
		       echo $USER
		       echo $sudopass
                       ansible-playbook  -i hosts.yml  --extra-vars"ansible_user="${USER}" --extra-vars ansible_password="${SUDOPASS}" \
		       --extra-vars ansible_ssh_common_args='"-o StrictHostKeyChecking=no -o ServerAliveInterval=30"' \
		       deploy.yml
                       '''
                   }
               } 
            }
          }
      }
    
    post {
		always {
			script {
				slackNotifier currentBuild.result
			}
		}  
    }
  }
