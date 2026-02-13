pipeline {
	agent {
        	label "${env.EXT_PIPELINE_AGENT}"
    	}

 	parameters {
		booleanParam(name:'RUN_TESTS',defaultValue:false,description:'Run tests?')
		string(name: 'PIPELINE_BRANCH', defaultValue: 'develop',description: 'Branch to build')
 		choice(
 			name: 'DEPLOYMENT_ENVIRONMENT',
 			choices: ['qa', 'sit', 'full', 'prod'],
 			description: 'Where should this feature branch deploy? (Only applies to feature/* branches)'
 		)
 	}

 	environment {
 		//JWT_KEY = credentials('ccr-salesforce-ssl-key')
		CLIENT_ID_FULL = credentials('ccr-salesforce-full-client')
		CLIENT_ID_PROD = credentials('ccr-salesforce-prod-client')
		//CLIENT_ID_QA = credentials('ccr-salesforce-qa-client')
		CLIENT_ID_SIT = credentials('ccr-salesforce-sit-client')
 	}

 	stages {

		
		stage('Authenticate to Salesforce') {
			steps {
				script {
					withCredentials(
                  [
                      usernamePassword(
                          credentialsId:'ccr-salesforce-qa-client',
                          usernameVariable:'CLIENT_ID_QA',
                          passwordVariable:'CLIENT_ID_PSW'
                      ),
                      string(
                          credentialsId:'ccr-salesforce-ssl-key',
                          variable:'JWT_KEY'                      
                      )
                  ]
                  ) {
					sh "echo 'CLIENT_ID_PSW......:${env.CLIENT_ID_PSW}'"
					sh "echo 'CLIENT_ID_QA......:${env.CLIENT_ID_QA}'"
					sh "echo 'JWT_KEY....:${env.JWT_KEY}'"
					def INSTANCE_URL = 'https://centralcustomerregistry--ccrqa1.sandbox.my.salesforce.com'
					
					/*
					sh """
					sfdx auth:jwt:grant \
					--clientid $env.CLIENT_ID_PSW \
					--jwt-key-file ./.assets/server.key \
					--username $env.CLIENT_ID_QA \
					--instanceurl $INSTANCE_URL \
					--setdefaultusername
							"""
					*/

					sh '''
						echo "-----BEGIN RSA PRIVATE KEY-----" > server.key
						echo "$env.JWT_KEY" >> server.key
						echo "-----END RSA PRIVATE KEY-----" >> server.key
						chmod 600 server.key
						cat server.key

					'''
					
					sh """
					sfdx auth:jwt:grant \
					--client-id $env.CLIENT_ID_PSW \
					--jwt-key-file server.key \
					--username $env.CLIENT_ID_QA \
					--instance-url $INSTANCE_URL \
					--set-default
							"""
					}
				} //script
			}//steps
 		}//stage

 	}

 	post {

		always {
			sh 'rm -f server.key'
		}
		
 		success {
 		echo "Deployment successful for branch ${env.PIPELINE_BRANCH}"
 		}
 		failure {
 			echo "Deployment failed for branch ${env.PIPELINE_BRANCH}"
 		}
 	}
}
