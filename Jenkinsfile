
library('cloud-jenkinsfile@master')
library('sre-jenkinsfile@master')


def account_numbers = getAwsAccounts("cloud_custodian","true")
def accounts = [:]

pipeline {
  agent {
    docker {
      image 'cloud-custodian:03-30-2020-2'
      label 'ec2-spot'
	  	registryUrl 'https://967877283425.dkr.ecr.us-east-1.amazonaws.com'
      registryCredentialsId 'ecr:us-east-1:jenkins-ecr'
    }
  }
	 parameters {
		 	
			// Account list are built based on the following boolean values.
			booleanParam(name: 'sandbox', defaultValue: true, description: 'deploy to sandbox')
  	}
	stages {
		stage ("code checkout") {
			steps {
				
				// checkout scm 
				git branch: 'master', credentialsId: 'cb4c203a-eaa9-4d25-9895-d175e7858d76', url: 'git@github.com:HBOCodeLabs/aws-cloud-custodian.git'			
				echo "job parameters ${params}"
				echo "job env ${env.BRANCH_NAME}"
				echo "job env ${GIT_BRANCH}"
				// git branch: 'sgadupu/new-accounts', credentialsId: 'd2db67f5-c452-4b68-92eb-2c6c782de453', url: 'git@github.com:HBOCodeLabs/DP-AWS-Cloud-Custodian.git'			
			}
		}
		stage('building account list') {
			steps{
				script {
					params.each { account,account_value ->
					  if ("${account_value}" == "true") {
							account_number = account_numbers[account]
							accounts.put(account,account_number)
						}
					}
				}
			}	
		}
		stage("Run Cloud custodian"){
			steps {
				script {
					run_cloud_custodian(accounts)
				}
			}
		}
		
		stage("deploy Cloud custodian"){
			input {
					message "Should deploy custodian policy"
					ok "Yes, we should."
			}
			steps {
				script {
					delete_cloud_custodian(accounts)
					run_cloud_custodian(accounts,false)
				}
			}
		}
}
	post {
		always {
			deleteDir()	
		}
		success {
			echo 'custodian policies successfully deployed'
		}
		failure {
			echo 'custodian policies deployment failed'
		}
	}
}
