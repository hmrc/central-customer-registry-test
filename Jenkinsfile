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
                SF_NPM_REGISTRY = "https://artefacts.tax.service.gov.uk/artifactory/api/npm/npmjs"
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
					sh """
echo $PATH
which node
ls -l /home/jenkins/.local/bin
							"""
					}
				} //script
			}//steps
 		}//stage

 	}

 	post {

		
 		success {
 		echo "Deployment successful for branch ${env.PIPELINE_BRANCH}"
 		}
 		failure {
 			echo "Deployment failed for branch ${env.PIPELINE_BRANCH}"
 		}
 	}
}
