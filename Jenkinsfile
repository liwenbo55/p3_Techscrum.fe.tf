// Jenkinsfile for Techscrum project (by Lawrence)
pipeline {
    agent any
    
    parameters {
        // Choose an environment to deploy frond-end resources: 'dev', 'uat', or 'prod'.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment.')

        // Apply or destroy resources
        choice(choices: ['deploy', 'destroy'], name: 'Operation', description: 'Deploy or destroy resources.')

        // Plan is used for gengrating plan file. Apply is used to deploy or destroy resources.
        choice(choices: ['plan','apply'], name: 'plan_apply', description: 'Plan is used for gengrating plan file. Apply is used to deploy or destroy resources.')
    }
    
    stages {
      stage('Check out code'){
        steps{
          git branch:'master', url:'https://github.com/liwenbo55/p3_Techscrum.tf.fe.git'                
        }
      }

      stage('IaC') {
        steps{
          script {
            withCredentials([
              [$class: 'AmazonWebServicesCredentialsBinding', 
               credentialsId: 'lawrence-jenkins-credential']
            ]){
              dir('app/techscrum_fe'){
                  
                  if (params.Environment in ['dev', 'uat', 'prod']) {
                      echo "Deploying front-end resources for ${params.Environment} environment."
                      
                      // Echo terraform vision
                      sh 'terraform --version'

                      // Terraform init
                      sh "terraform init -reconfigure -backend-config=backend_${params.Environment}.conf"

                      // Terraform plan
                      if (params.Operation == 'deploy') {
                          sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan"
                      } else if (params.Operation == 'destroy') {
                          sh "terraform plan -var-file=${params.Environment}.tfvars -out=${params.Environment}_${params.Operation}_plan -destroy"
                      } else {
                          error "Invalid Operation: ${params.Operation}."
                      }
                                          
                      // Terraform actions
                      if (params.plan_apply == 'apply') {
                          // Terraform APPLY
                          sh "terraform apply '${params.Environment}_${params.Operation}_plan'"
                        } 
                      
                      sh 'ls -la'
                      
                      // Generate a readable plan file
                      sh "terraform show -no-color ${params.Environment}_${params.Operation}_plan > ${params.Environment}_${params.Operation}_plan.txt "                      
                          
                    } else {
                        error "Invalid environment: ${params.Environment}."
                    }                 

              }
            }
          }
        }
      }
    }
    post {
        success {
            echo "Front-end: ${params.Environment}--${params.Operation}--${params.plan_apply} has succeeded."
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end terraform pipeline for ${params.Environment} environment succeeded.",
                body: 
                    """
                    Pipeline succeeded. \nEnvironment: ${params.Environment}. \nOperation: ${params.Operation}--${params.plan_apply}. \nPlease check the plan file.
                    """,
                attachLog: false,
                attachmentsPattern: "**/${params.Environment}_${params.Operation}_plan.txt"
            )
        }

        failure {
            echo "Front-end: ${params.Environment}--${params.Operation}--${params.plan_apply} has failed."
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end terraform pipeline for ${params.Environment} environment failed.",
                body: 
                    """
                    Pipeline failed.\nEnvironment: ${params.Environment}. \nOperation: ${params.Operation}--${params.plan_apply}. \nPlease check logfile for more details.
                    """,
                attachLog: true
            )
        }
    }
}
