# Mulesoft Jenkin Pipeline

# MAKE SURE YOU ARE USING THE LATEST PARENT POM (VERSION 1.2.6)

pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }
    tools { 
        maven 'Maven 3.3.9' 
        jdk 'jdk17' 
    }
    // Define parameters for the pipeline
    parameters {
      booleanParam(name:'RUN_TESTS',defaultValue:true,description:'Run tests stage?')
      choice(name:'PIPELINE_BRANCH',defaultValue:'develop',choices:['develop','sit','uat','pre','prod'],description:'Branch to build')
      choice(name:'DEPLOYMENT_ENVIRONMENT',choices:['develop','sit','uat','pre','prod'],description:'Select environment')
      string(name:'USERNAME',defaultValue:'admin',description:'User name for deployment')
      password(name:'PASSWORD',defaultValue:'admin',description:'Password for deployment')
     }

    environment {
      SECURE_KEY = ""
      BUILD_TOOL = 'Maven'
      MAVEN_HOME = tool name:'Maven', type:'maven'
      SETTING_PATH=
      ANYPOINT_LASTMILE='true'
      ANYPOINT_ENDPOINT = ''
      UPDATE_STRATEGY =''
      ANYPOINT_VCORES = '1'
      ANYPOINT_REPLICAS= ''
      MULE_CONNECTED_APP = credentials('ccr-connected-app')
      MULE_EE_NEXUS = credentials('ccr-mule-ee-nexus')
      MULE_ANYPOINT_ENV_CREDENTIALS =credentials('ccr-mule-anypoint')
      APP_NAME = "my-mule-app"
      BRANCH_NAME = params.PIPELINE_BRANCH
    }
    
  
    stages {
    
      stage('Echo Parameters') {
            steps{
                script{
                    //Access parameters using the 'params' object
                    @echo off
                    echo "PATH = ${SETTING_PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "Username:${params.USERNAME}"
                    echo "Debug Mode:${params.DEBUG_MODE}"
                    echo "Environment:${params.ENVIRONMENT}"
                    echo "Password is set ${params.PASSWORD?.length() > 0}"                    
                    echo GIT_COMMIT %GIT_COMMIT% 
                    echo GIT_BRANCH %GIT_BRANCH%
                    echo GIT_LOCAL_BRANCH %GIT_LOCAL_BRANCH%
                    echo GIT_PREVIOUS_COMMIT %GIT_PREVIOUS_COMMIT%
                    echo GIT_PREVIOUS_SUCCESSFUL_COMMIT %GIT_PREVIOUS_SUCCESSFUL_COMMIT%
                    echo GIT_URL %GIT_URL%
                    echo GIT_URL_N - %GIT_URL_N%
                    echo GIT_AUTHOR_NAME %GIT_AUTHOR_NAME%
                    echo GIT_COMMITTER_EMAIL %GIT_COMMITTER_EMAIL%
                }
            }
      }//stage
    }//stages

}//pipeline
