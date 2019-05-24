 pipeline {
    agent {
        docker {
            image 'digitalonus/terraform_hub:0.11.10'
        }
    }
    environment {
        DIGITALOCEAN_TOKEN= credentials('DO_TOKEN')
        TOKEN = credentials('gh-token')
    }
    triggers {
         pollSCM('H/5 * * * *')
    }
    stages {
        stage('init'){
          when { expression { env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
          steps{
            sh 'cd terraform && terraform init -input=false'    
          }
        }
        stage('validate'){
          when { expression { env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
          steps{
            sh 'cd terraform && terraform validate'    
          }
        }
        stage('plan'){
            when { expression{ env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                sh 'cd terraform && terraform plan -out=plan -input=false'
                input(message: "Do you want to create a PR to apply this plan?", ok: "yes")
            }
        }
        stage('Generate PR'){
            when { expression{ env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/  } }
            steps{
                createPR "mons3rrat", "PR Created Automatically by Jenkins", "master", env.BRANCH_NAME, "mons3rrat"
                //slackSend baseUrl: readProperties.slack, channel: '#dou-workshop', color: '#00FF00', message: "Please review and approve PR to merge changes to dev branch : https://github.com/mons3rrat/tf_pipeline_workshop/pulls"
            }
        }
        stage('apply') {
            when { expression{ env.BRANCH_NAME ==~ /master.*/} }
            steps {
                sh 'cd terraform && terraform apply -input=false plan'
            }
        }
        stage('destroy') {
            when { expression{ env.BRANCH_NAME ==~ /master.*/ } }
            steps {
                sh 'cd terraform && terraform destroy -force -input=false'
            }
        }
    }
    post {
      success {
          echo 'success'
      }
      failure {
           echo 'FAILED'
      }
    }
}

def createPR (user, title, tobranch, frombranch, org){
  def COMMIT_MESSAGE = sh(script:'git log -1 --pretty=%B', returnStdout: true).trim()
  sh "if [ ! -d ~/.config ]; then $(which sudo) mkdir ~/.config;fi"
  sh 'echo "github.com:" >> ~/.config/hub'
  sh "echo \"- user: ${user}\" >> ~/.config/hub"
  sh "echo \"  oauth_token: ${env.TOKEN}\" >> ~/.config/hub"
  sh 'echo "  protocol: https" >> ~/.config/hub'
  try {
      sh "git checkout ${env.BRANCH_NAME}"
      sh "hub pull-request -m \"${title}\n ${COMMIT_MESSAGE} \n From Jenkins job: ${env.BUILD_URL} \" -b ${org}:${tobranch} -h ${org}:${frombranch}"
  }catch(Exception e) {
      echo "PR already created"
  }
}

