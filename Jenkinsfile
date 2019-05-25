 pipeline {
    agent {
        docker {
            image 'digitalonus/terraform_hub:0.11.10'
        }
    }
    environment {
        DIGITALOCEAN_TOKEN= credentials('DO_TOKEN')
        TOKEN = credentials('gh-token')
        GITUSER = credentials('git_user')
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
        stage('plan and create PR'){
            when { expression{ env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                script {
                    def COMMIT_MESSAGE = sh(script:'git log -1 --pretty=%B', returnStdout: true).trim()
                    sh "cd terraform && terraform plan -out=plan -input=false"
                    input(message: "Do you want to create a PR to apply this plan?", ok: "yes")
                    httpRequest authentication: 'git_user', contentType: 'APPLICATION_JSON_UTF8', httpMode: 'POST', requestBody: """{ "title": "PR Created Automatically by Jenkins", "body": "${COMMIT_MESSAGE} \n From Jenkins job: ${env.BUILD_URL} ", "head": "mons3rrat:${env.BRANCH_NAME}", "base": "master"}""", url: "https://api.github.com/repos/mons3rrat/tf_pipeline_workshop/pulls"
                }
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
