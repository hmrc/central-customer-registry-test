
//Mulesoft Jenkin Pipeline

//MAKE SURE YOU ARE USING THE LATEST PARENT POM (VERSION 1.2.6)
//ccr-anypoint-dev-client  -anypoint dev env credentials
//ccr-anypoint-pre-client
//ccr-anypoint-prod-client
//ccr-anypoint-sit-client
//ccr-anypoint-uat-client
//ccr-decryption-dev    -key for derypting secure propertis
//ccr-decryption-pre
//ccr-decryption-prod
//ccr-decryption-sit
//ccr-decryption-uat

pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }
    
    // Define parameters for the pipeline
    parameters {
      booleanParam(name:'RUN_TESTS',defaultValue:true,description:'Run tests stage?')
      choice(name:'PIPELINE_BRANCH',choices:['develop','sit','uat','pre','prod'],description:'Branch to build')
      choice(name:'DEPLOYMENT_ENVIRONMENT',choices:['develop','sit','uat','pre','prod'],description:'Select environment')
      string(name:'USERNAME',defaultValue:'',description:'User name for deployment')
      password(name:'PASSWORD',defaultValue:'',description:'Password for deployment')
     }

    environment {
      BUILD_TOOL = 'Maven'
      ANYPOINT_LASTMILE = 'true'
      ANYPOINT_ENDPOINT = 'anypoint.mulesoft.com'
      UPDATE_STRATEGY = 'rolling'
      ANYPOINT_VCORES = '.5'
      ANYPOINT_REPLICAS= '1'
      MULE_CONNECTED_APP = credentials('ccr-connected-app')
      MULE_EE_NEXUS = credentials('ccr-mule-ee-nexus')      
      GITHUB_CREDENTIALS =credentials('github-hmrc-read-only-usert')
      BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
      APP_NAME = "my-mule-app"
     
    }
    
  
    stages {
    
      stage('Echo_Parameters') {
            steps{
                
                    //Access parameters using the 'params' object
                    
                    echo "PATH = ${SETTING_PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "Username:${params.USERNAME}"
                    echo "Debug Mode:${params.RUN_TESTS}"
                    echo "Environment:${params.DEPLOYMENT_ENVIRONMENT}"
                    echo "Password is set ${params.PASSWORD?.length() > 0}"   
                    ecgo "anypoint client id :${env.MULE_ANYPOINT_ENV_CREDENTIALS.ANYPOINT_CLIENT_ID}"                 
                    echo 'GIT_COMMIT '+ ${env.GIT_COMMIT}
                    //echo GIT_BRANCH %GIT_BRANCH%
                    echo 'Pulling...' + ${env.BRANCH_NAME}
                    echo 'Pulling...' + ${params.PIPELINE_BRANCH}  
                                     
                    //echo GIT_PREVIOUS_COMMIT %GIT_PREVIOUS_COMMIT%
                    //echo GIT_PREVIOUS_SUCCESSFUL_COMMIT %GIT_PREVIOUS_SUCCESSFUL_COMMIT%
                    //echo GIT_URL %GIT_URL%
                    //echo GIT_URL_N - %GIT_URL_N%
                    //echo GIT_AUTHOR_NAME %GIT_AUTHOR_NAME%
                    //echo GIT_COMMITTER_EMAIL %GIT_COMMITTER_EMAIL%
                
            }
      }//stage

      //extract asset parameters from pom.xml
      stage('Parse_POM') {
            steps {
                script {

                      def groupId = sh (
                      script:"mvn  --settings .mvn/settings.xml help:evaluate -Dexpression=project.groupId -q -DforceStdout",
                      returnStdout:true).trim()
                       
                      echo "groupId :${groupId}"
                      
                      def assetId =sh (                    
                      script:"mvn  --settings .mvn/settings.xml help:evaluate -Dexpression=project.artifactId -q -DforceStdout",
                      returnStdout:true).trim()                    
                      echo "assetId :${assetId}"
                      
                      dev version = sh(
                      script:"mvn  --settings .mvn/settings.xml help:evaluate -Dexpression=project.version -q -DforceStdout",
                      returnStdout:true).trim()
                      echo "version :${version}"
                      
                      environment {
                        assetGroup = $groupId
                        assetId = $assetId
                        version = $version
                      }
                 }
            }
      }//stage

      //Determine the Maven profile based on assetId
      stage('Determine_Maven_Profile') {
            steps {
                script {
                    if (contains(${env.assetId} ,"xapi") || contains(${env.assetId},"xpap")) {
                        environment {profile = "-Pexperience"}
                        echo "profile is set to ${env.profile}"
                    } else {
                        environment {profile = "-Psystem"}
                        sh echo "profile is set to ${env.profile}"
                  }//else
                }
               }
      }//stage
    
      
      //Check if asset exists
 
      stage('Check_Asset_in_Exchange') {
            steps {
                script {
                def token =''
                if(!contains(env.assetVersion, "SNAPSHOT")){
                    //Step 1:Get Anypoint Access Token
                    withCredentials([string(credentialsId:env.MULE_CONNECTED_APP, variable:'MULE_CONNECTED_APP_CLIENT_ID', passwordVariable:'MULE_CONNECTED_APP_CLIENT_SECRET')]) 
                    withCredentials([string(credentialsId:env.MULE_EE_NEXUS, variable:'MULE_EE_NEXUS_USER', passwordVariable:'MULE_EE_NEXUS_PASSWORD')]) 
 
                    def tokenResponse = sh(returnStdout:true,script:"""curl -s -X POST https://anypoint.mulesoft.com/accounts/api/v2/oauth2/token \\
                        -H 'Content-Type:application/json' \\
                        -d '{
                            "grant_type":"client_credentials",
                            "client_id":\\"${MULE_CONNECTED_APP_CLIENT_ID}\\",
                            "client_secret":\\"${MULE_CONNECTED_APP_CLIENT_SECRET}\\"                            
                        }'
                        """).trim()

                    token = new groovy.json.JsonSlurper().parseText(tokenResponse).access_token
                    if (!token) {
                          error "Failed to get Anypoint access token"
                    }
                 }//endif

                  //Step 2:GraphQL query to check asset
                  def graphqlQuery = """
                  {
                    assets(filter:{
                      groupId:\\"${env.assetGroup}\\",
                      assetId:\\"${env.assetId}\\",
                      version:\\"${env.version}\\"
                    }) {
                      total
                      results {
                        assetId
                        version
                      }
                    }
                  }
                  """

                  def assetResponse = sh(returnStdout:true,script:"""
                      curl -s -X POST https://anypoint.mulesoft.com/exchange/api/v2/graphql \\
                      -H "Authorization:Bearer ${token}" \\
                      -H "Content-Type:application/json" \\
                      -d '{"query":"${graphqlQuery.replaceAll("\\n", " ")}"}'
                      """
                  ).trim()

                  def assetData = new groovy.json.JsonSlurper().parseText(assetResponse)
                  def total = assetData.data.assets.total
                  if (total > 0) {
                      environment { assestFound = 'true' }
                      echo "Asset exists in Exchange"
                  } else {
                       echo "Asset does NOT exist in Exchange"
                       environment { assestFound = 'false' }
                  }
                }
            }//steps
        }
      
      
      //Publish to Anypoint Exchange
      stage('Publish_to_Anypoint_Exchange')
      {
        steps { 
          script {
            withCredentials([string(credentialsId:env.MULE_CONNECTED_APP, variable:'MULE_CONNECTED_APP_CLIENT_ID', passwordVariable:'MULE_CONNECTED_APP_CLIENT_SECRET')]) 
            withCredentials([string(credentialsId:env.MULE_EE_NEXUS, variable:'MULE_EE_NEXUS_USER', passwordVariable:'MULE_EE_NEXUS_PASSWORD')]) 
   
            if( ${env.assestFound} != "true")
              {
                sh ("mvn --settings .mvn/settings.xml --batch-mode --update-snapshots deploy -Pcp_us -DskipTests ${env.profile} ",
                returnStdout:true).trim()
              }
            }
         }
      }//stage


      stage('munitTests') {
          when {
                expression { return params.RUN_TESTS }
            }
            steps {
                sh "${MAVEN_HOME}/bin/mvn test"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
      }//stage

      //Environment-Based Deployment
      stage('Environment_Based_Deployment') {
      
            steps {         
                
                script {
                    def MULE_ANYPOINT_ENV_CREDENTIALS = credentials('ccr-anypoint-dev-client')
                    def SECURE_KEY = credentials('ccr-decryption-dev')
                    withCredentials([string(credentialsId:env.MULE_CONNECTED_APP, variable:'MULE_CONNECTED_APP_CLIENT_ID', passwordVariable:'MULE_CONNECTED_APP_CLIENT_SECRET')]) 
                    withCredentials([string(credentialsId:env.MULE_EE_NEXUS, variable:'MULE_EE_NEXUS_USER', passwordVariable:'MULE_EE_NEXUS_PASSWORD')]) 
                    withCredentials([string(credentialsId:env.MULE_ANYPOINT_ENV_CREDENTIALS, variable:'ANYPOINT_CLIENT_ID_DEV', passwordVariable:'ANYPOINT_CLIENT_SECRET_DEV')]) 

                    println("MULE_CONNECTED_APP_CLIENT_ID :" + MULE_CONNECTED_APP_CLIENT_ID)
                    
                    def envName = ${params.DEPLOYMENT_ENVIRONMENT}
                    if (envName == "develop")
                    {
                      echo "Deploying to Development environment:${envName}"
                      def publicUrl=""
                      if ( contains(${env.assetId} ,"xapi") || contains(${env.assetId}."xpap" )) {
                        publicUrl="-Danypoint.publicUrl=true"
                      } else{
                        publicUrl="-Danypoint.publicUrl=false"
                      }
                      
                      def vanitydomain ="-https://dev-ccr.api.mulesoft.hmrc.gov.uk/${env.ANYPOINT_ENDPOINT},https://dev-ccr.api.mulesoft.hmrc.gov.uk/${env.ANYPOINT_ENDPOINT}${env.ANYPOINT_ENDPOINT}\${publicUrl} -DmuleDeploy -DskipTests -Pcp_us ${env.profile}"
                      echo "vanitydomain:${vanitydomain}"
                      sh (script:'mvn --settings .mvn/settings.xml --batch-mode --update-snapshots deploy -DskipTest \
                      -Danypoint.lastMile=${env.ANYPOINT_LASTMILE} \
                      -DreleaseChannel=LTS \
                      -Danypoint.properties.key=${SECURE_KEY} \
                      -Danypoint.clientSecret=${env.ANYPOINT_CLIENT_SECRET_DEV} \
                      -Danypoint.clientId=${env.ANYPOINT_CLIENT_ID_DEV} \
                      -Danypoint.properties.env=dev \
                      -Danypoint.properties.envName=-dev \
                      -Danypoint.skipDeploymentVerification=true \
                      -Danypoint.replicas=${env.ANYPOINT_REPLICAS} \
                      -Danypoint.vCores=${env.ANYPOINT_VCORES}  \
                      -Danypoint.spaceName=ucr-privatespace-nonprod \
                      -Danypoint.environment=DEV \
                      -Danypoint.updateStrategy=${env.UPDATE_STRATEGY} \
                      -Danypoint.connectedAppId=env.MULE_CONNECTED_APP_CLIENT_ID \
                      -Danypoint.connectedAppSecret=env.MULE_CONNECTED_APP_CLIENT_SECRET \
                      -Danypoint.vanityDomain=${vanitydomain}'
                      ,returnStdout:true).trim()
                    }
                    else if (envName == "sit")
                    {
                        echo "Deploying to Staging environment"
                    }
                    else if (envName == "uat")
                    {
                        echo "Deploying to Staging environment"
                    } else {

                       echo "Unknown environment:${envName}"
                   }//switch
               }//script
           }//step
      }
      
    
    }//stages
    
    
   

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
