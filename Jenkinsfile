 def builduser = null
 pipeline {
    agent {
        docker {
            image 'digitalonus/terraform_hub:0.11.10'
        }
    }
    environment {
        DIGITALOCEAN_TOKEN = sh(script:'vault kv get -field=token workshop/mons3rrat/digitalocean > /dev/null 2>&1')
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
                    def jobName = "tf_pipeline_workshop"
                    wrap([$class: 'BuildUser']) {builduser = env.BUILD_USER_ID}
                    sh "cd terraform && terraform plan -out=plan -input=false"
                    input(message: "Do you want to create a PR to apply this plan?", ok: "yes")
                    httpRequest authentication: 'git_user', contentType: 'APPLICATION_JSON_UTF8', httpMode: 'POST', requestBody: """{ "title": "PR Created Automatically by Jenkins", "body": "From Jenkins job: ${env.BUILD_URL}", "head": "${builduser}:${env.BRANCH_NAME}", "base": "master"}""", url: "https://api.github.com/repos/${builduser}/${jobName}/pulls"
                }
            }
        }
        stage('apply') {
            when { expression{ env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
            steps {
                sh 'cd terraform && terraform apply -input=false plan'
            }
        }
        stage('destroy') {
            when { expression{ env.BRANCH_NAME ==~ /dev.*/ || env.BRANCH_NAME ==~ /PR.*/ || env.BRANCH_NAME ==~ /feat.*/ } }
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
