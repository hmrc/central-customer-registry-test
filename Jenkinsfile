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

 		stage('Identify Branch & Target Org') {
 			steps {
 				script {
					echo "Branch--PIPELINE_BRANCH--: ${env.PIPELINE_BRANCH}"
					echo "Branch- DEPLOYMENT_ENVIRONMENT---: ${env.DEPLOYMENT_ENVIRONMENT}"
 					if (env.PIPELINE_BRANCH == 'main') {
 						//INSTANCE_URL = 'https://centralcustomerregistry.my.salesforce.com'
 						//TEST_LEVEL = 'RunAllTestsInOrg'
 						//FORCE_VALIDATION = true
 						//BASE_BRANCH = 'main'
						//CLIENT_ID = credentials('ccr-salesforce-prod-client')
						//DEPLOYMENT_USER = 'release.deployment@hmrc.com'
 					}

 					else if (env.PIPELINE_BRANCH == 'develop') {
 						env.INSTANCE_URL = 'https://centralcustomerregistry--ccrsit2.sandbox.my.salesforce.com'
 						env.TEST_LEVEL = 'RunLocalTests'
 						env.FORCE_VALIDATION = false
 						env.BASE_BRANCH = 'develop'
						env.CLIENT_ID = credentials('ccr-salesforce-sit-client')
						env.DEPLOYMENT_USER = 'release.deployment@hmrc.com.ccrsit2'
 					}

 					else if (env.PIPELINE_BRANCH.startsWith('release/')) {
 						env.INSTANCE_URL = 'https://centralcustomerregistry.my.salesforce.com'
 						env.TEST_LEVEL = 'RunAllTestsInOrg'
 						env.FORCE_VALIDATION = true
 						env.BASE_BRANCH = 'main'
						//env.CLIENT_ID = credentials('ccr-salesforce-prod-client')
						//env.DEPLOYMENT_USER = 'release.deployment@hmrc.com'
 					}

 					else if (env.PIPELINE_BRANCH.startsWith('feature')) {
						echo "Branch: Feature"
 						if (params.DEPLOYMENT_ENVIRONMENT == 'qa') {
 							env.INSTANCE_URL = 'https://centralcustomerregistry--ccrqa1.sandbox.my.salesforce.com'
 							env.TEST_LEVEL = 'NoTestRun'
 							env.FORCE_VALIDATION = false
 							env.BASE_BRANCH = 'develop'
							//env.CLIENT_ID = env.CLIENT_ID_QA
							//env.DEPLOYMENT_USER = 'release.deployment@hmrc.com.ccrqa1'

 						} else if (params.DEPLOYMENT_ENVIRONMENT == 'sit') {
 							env.INSTANCE_URL = 'https://centralcustomerregistry--ccrsit2.sandbox.my.salesforce.com'
 							env.TEST_LEVEL = 'RunLocalTests'
 							env.FORCE_VALIDATION = false
 							env.BASE_BRANCH = 'develop'
							//env.CLIENT_ID = credentials('ccr-salesforce-sit-client')
							//env.DEPLOYMENT_USER = 'release.deployment@hmrc.com.ccrsit2'

 						} else if (params.DEPLOYMENT_ENVIRONMENT == 'full') {
 							env.INSTANCE_URL = 'https://centralcustomerregistry--ccrfull.sandbox.my.salesforce.com'
 							env.TEST_LEVEL = 'NoTestRun'
 							env.FORCE_VALIDATION = false
 							env.BASE_BRANCH = 'develop'
							//env.CLIENT_ID = credentials('ccr-salesforce-full-client')
							//env.DEPLOYMENT_USER = 'release.deployment@hmrc.com.ccrfull'

						} else if (params.DEPLOYMENT_ENVIRONMENT == 'prod') {
 							//INSTANCE_URL = 'https://centralcustomerregistry.my.salesforce.com'
 							//TEST_LEVEL = 'RunAllTestsInOrg'
 							//FORCE_VALIDATION = true
 							//BASE_BRANCH = 'main'
							//CLIENT_ID = credentials('ccr-salesforce-prod-client')
							//DEPLOYMENT_USER = 'release.deployment@hmrc.com'

 						} else {
 							error "Feature branch requires selecting a deployment target."
 						}
 					}

 					else {
 						error "Unknown branch type: ${env.DEPLOYMENT_ENVIRONMENT}"
 					}

 					echo "Branch: ${env.PIPELINE_BRANCH}"
 					echo "Deploying to: ${env.DEPLOYMENT_ENVIRONMENT}"
 					echo "Test Level: ${env.TEST_LEVEL}"
 					echo "Force Validation: ${env.FORCE_VALIDATION}"
 					echo "Delta Base Branch: ${env.BASE_BRANCH}"
					echo "User Name: ${env.DEPLOYMENT_USER}"
					//echo "Branch***: ${env.CLIENT_ID}"
 				}
 			}
 		}

 		stage('Checkout Source') {
 			steps {
 				checkout scm
 			}
 		}

		/*
 		stage('Install Tools') {
 			steps {
 				sh '''
 				npm install sfdx-cli --global
 				npm install sfdx-git-delta --global
 				sfdx --version
 				'''
 			}
 		}
		*/
		
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

 		stage('Generate Delta Package') {
 			steps {
 				sh '''
 					echo "Generating delta from ${env.BASE_BRANCH} to ${env.PIPELINE_BRANCH}"
 					sgd --to HEAD --from origin/${env.BASE_BRANCH} --output delta
 				'''
 			}
 		}

 		stage('Deploy Delta to Salesforce') {
 			steps {
 				script {

					def validationFlag = env.FORCE_VALIDATION ? "-c" : ""

 					def deployCmd = """
 					sfdx force:source:deploy \
 					-x delta/package.xml \
 					-u ${env.DEPLOYMENT_USER} \
 					-l ${env.TEST_LEVEL} \
 					${validationFlag} \
 					-w 30
 					"""

 					echo "Final Deployment Command:"
 					echo deployCmd

 					sh deployCmd
 				}
 			}
 		}
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
