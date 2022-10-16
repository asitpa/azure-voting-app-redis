pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
              echo "$GIT_BRANCH"
}
        }
        stage('Docker Build') {
         steps {
            pwsh(script: 'docker images -a')
            pwsh(script: """
               cd azure-vote/
               docker images -a
               docker build -t jenkins-pipeline .
               docker images -a
               cd ..
            """)
         }
      }
         stage('Start test app') {
         steps {
            pwsh(script: """
               docker-compose up -d
           
            """)
         }
         post {
            success {
               echo "App started successfully :)"
            }
            failure {
               echo "App failed to start :("
            }
         }
      }
      stage('Run Tests') {
         steps {
            pwsh(script: """
               pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop test app') {
         steps {
            pwsh(script: """
               docker-compose down
            """)
         }
      }
         stage('Container Scanning') {
         parallel {
            stage('Run Anchore') {
               steps {
                  pwsh(script: """
                     Write-Output "blackdentech/jenkins-course" > anchore_images
                  """)
                  
               }
            }
            stage('Run Trivy') {
               steps {
                  sleep(time: 30, unit: 'SECONDS')
                   pwsh(script: """
                    ubuntu run trivy image --format template --template "@contrib/html.tpl" -o "trivy-report${BUILD_NUMBER}.html"  python:3.4-alpine --security-checks vuln
                   """)
               }
            }
        
      }
                  stage('Deploy to Prod') {
         environment {
            ENVIRONMENT = 'prod'
         }
         steps {
            echo "Deploying to ${ENVIRONMENT}"
            acsDeploy(
               azureCredentialsId: "AzureCred",
               configFilePaths: "**/*.yaml",
               containerService: "prod_kub | AKS",
               resourceGroupName: "prod_kub",
               sshCredentialsId: ""
            )
         }
         }
      }
    }
}
