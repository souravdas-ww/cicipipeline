#!groovy

/**
 *
 * Author:    Naresh Manthrabuddi
 * Created:   24.07.2019
 *
 **/

pipeline {
  agent any

	stages {
	    stage('Pipeline gets Started with log read') {
			steps {
			   echo 'OASIS Project Pipeline Started'
			}
	    }
	}
}
node {

	def v_environment = ""
	def v_buildMechanism =""
	def v_muleRuntimeEnvironment =""
	def v_workers = ""
	def v_cores = ""
	def v_appExists= ""
	def v_applicationName = ""
	def v_anypointCredentialID = ""
	def v_anypointOrganization = ""
	def v_AnypointEnvironment = ""
	def v_muleEvn = ""
	def v_encryptKey = ""
	def v_artifactId = ""
	def v_version = ""
	def v_package = ""
	def v_downloadFilePath = ""
	def v_jenkinsLogPath = ""

	properties([
     parameters([
       choiceParam(
         choices: 'DEV\nSIT\nUAT\nPROD',
            description: 'Please select the ENVIRONMENT for Deployment',
            name: 'ENVIRONMENT'
       ),
       choiceParam(
         choices: 'BUILD-MUNITS\nBUILD-MUNITS-SONAR\nBUILD-MUNITS-SONAR-RELEASE',
            description: 'Please select the Build Mechanism',
            name: 'BUILD_MECHANISM'
       ),
       choiceParam(
         choices: '4.1.3\n4.1.4\n4.1.5\n4.2.0',
            description: 'Please select mule runtime version for Deployment?',
            name: 'MULE_RUNTIME_VERSION'
       ),
       choiceParam(
         choices: '1\n2\n3\n4\n5\n6\n7\n8',
            description: 'Please select the workerSize for Deployment?',
            name: 'WORKERS'
       ),
       choiceParam(
         choices: '0.1\n0.2\n1\n2\n4\n8\n16',
            description: 'Please select the vCores for Deployment?',
            name: 'VCORES'
       ),
       choiceParam(
         choices: 'YES\nNO',
            description: 'Is application existed on cloud?/Are you re-deploying application on Cloud?',
            name: 'APP_EXISTS'
       ),
       string(
		   name: 'APPLICATION_NAME',
		   defaultValue: 's-oasis-next-api',
		   description: 'Please enter the application name for CloudHub Deployment?'
	   ),
       choiceParam(
         choices: 'DEV_CREDENTIAL_ID\nSIT_CREDENTIAL_ID\nPROD_CREDENTIAL_ID',
            description: 'Please select the vCores for Deployment?',
            name: 'ANYPOINT_CREDENTIAL_ID'
       ),
       string(
		   name: 'ANYPOINT_ORGANIZATION',
		   defaultValue: 'Oasis Fashion',
		   description: 'Please provide CloudHub Anypoint Organization name to deploy?'
	   ),
	   choiceParam(
         choices: 'DEV\nSIT\nUAT\nPROD',
            description: 'Please select the Cloudhub Anypoint Environment to deploy?',
            name: 'ANYPOINT_ENVIRONMENT'
       )
     ])
    ])

	if(params.BUILD_MECHANISM == 'BUILD-MUNITS') {
		try{
			stage 'Code Build & MUnits Execution'
				UDF_BuildSourceCode()

			stage 'Notification'
				SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","success")

		} catch(error) {
			throw(error)
			SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")

		}
	} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR') {
		try{
			stage 'Code Build & MUnits Execution'
				UDF_BuildSourceCode()

			stage 'SonarQube Analysis'
				UDF_ExecuteSonarQubeRules()

			stage 'Notification'
				SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","success")

		} catch(error) {
			throw(error)
			SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
		}
	} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR-RELEASE') {
		try{
			stage 'Code Build & MUnits Execution'
				UDF_BuildSourceCode()

			stage 'SonarQube Analysis'
				UDF_ExecuteSonarQubeRules()

			stage 'DeployToCloudHub'
				UDF_DeployToCloudHub()

			stage 'Notification'
				SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
		} catch(error) {
			throw(error)
			SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
		}
	}
}

/*
BUILD - STAGE
This function provides the functionality to build your clode
*/
def UDF_BuildSourceCode()
{
	try	{
		echo 'Build is Starting'
		sh 'mvn -U install -DskipTests=true'
		echo 'Build Completed'
	}catch(error) {
		throw(error)
		SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
	}
}

/*
SONARQUBE - STAGE
This function provides functionality to do the SONAR Analysis
*/
def UDF_ExecuteSonarQubeRules()
{
	try{
		echo 'SonarQube Rules Execution started'
		withSonarQubeEnv('SonarServer-Local') {
			sh 'mvn sonar:sonar'
		}
		echo 'SonarQube Rules Execution Completed'
	} catch(error) {
		throw(error)
		SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
	}
}

/*
DEPLOY STAGE
This function provides functionality to deploy the application package(zip file) to CloudHub Runtime
*/

def UDF_DeployToCloudHub() {

	echo "###### Application Deployment Stage ######"

	v_environment = "${params.ENVIRONMENT}"
	v_buildMechanism ="${params.BUILD_MECHANISM}"
	v_muleRuntimeEnvironment ="${params.MULE_RUNTIME_VERSION}"
	v_workers = "${params.WORKERS}"
	v_cores = "${params.VCORES}"
	v_appExists= "${params.APP_EXISTS}"
	v_applicationName = "${params.APPLICATION_NAME}"
	v_anypointCredentialID = "${params.ANYPOINT_CREDENTIAL_ID}"
	v_anypointOrganization = "${params.ANYPOINT_ORGANIZATION}"
	v_AnypointEnvironment = "${params.ANYPOINT_ENVIRONMENT}"
	v_muleEvn = ""
	v_encryptKey = ""
	v_artifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
	v_version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
	v_package = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
	v_downloadFilePath = "${env.WORKSPACE}\\target\\${v_artifactId}-${v_version}-${v_package}.jar"
	v_jenkinsLogPath = "${JENKINS_HOME}\\jobs\\${v_applicationName}\\branches\\${JOB_BASE_NAME}\\builds\\${BUILD_NUMBER}\\log"

	echo "ENVIRONMENT is : ${v_environment}"
	echo "BUILD_MECHANISM is : ${v_buildMechanism}"
	echo "MULE_RUNTIME_VERSION is : ${v_muleRuntimeEnvironment}"
	echo "WORKERS : ${v_workers}"
	echo "VCORES : ${v_cores}"
	echo "APP_EXISTS is : ${v_appExists}"
	echo "APPLICATION_NAME is : ${v_applicationName}"
	echo "ANYPOINT_CREDENTIAL_ID is : ${v_anypointCredentialID}"
	echo "ANYPOINT_ORGANIZATION is : ${v_anypointOrganization}"
	echo "ANYPOINT_ENVIRONMENT is : ${v_AnypointEnvironment}"
	echo "Environment Workspace is : ${env.WORKSPACE}"
	echo "Download File Path is : ${v_downloadFilePath}"
	echo "Jenkins log path : ${v_jenkinsLogPath}"

	echo "BUILD_NUMBER :: ${BUILD_NUMBER}"
	echo "BUILD_ID :: ${BUILD_ID}"
	echo "BUILD_DISPLAY_NAME :: ${BUILD_DISPLAY_NAME}"
	echo "JOB_NAME :: ${JOB_NAME}"
	echo "JOB_BASE_NAME :: ${JOB_BASE_NAME}"
	echo "BUILD_TAG :: ${BUILD_TAG}"
	echo "EXECUTOR_NUMBER :: ${EXECUTOR_NUMBER}"
	echo "NODE_NAME :: ${NODE_NAME}"
	echo "NODE_LABELS :: ${NODE_LABELS}"
	echo "WORKSPACE :: ${WORKSPACE}"
	echo "JENKINS_HOME :: ${JENKINS_HOME}"
	echo "JENKINS_URL :: ${JENKINS_URL}"
	echo "BUILD_URL ::${BUILD_URL}"
	echo "JOB_URL :: ${JOB_URL}"

	if("${params.ENVIRONMENT}" == 'DEV') {

		v_applicationName = "dev-${params.APPLICATION_NAME}"
		v_muleEvn = 'dev'
		v_encryptKey = 'MULESOFT_DEV'

	} else if("${params.ENVIRONMENT}" == 'SIT') {

		v_applicationName="sit-${params.APPLICATION_NAME}"
		v_muleEvn = 'sit'
		v_encryptKey = 'MULESOFT_SIT'

	} else if("${params.ENVIRONMENT}" == 'PROD') {

		v_applicationName="${params.APPLICATION_NAME}"
		v_muleEvn = 'prod'
		v_encryptKey = 'MULESOFT_PROD'

	}

	if("${params.ANYPOINT_CREDENTIAL_ID}" == 'DEV_CREDENTIAL_ID' || "${params.ANYPOINT_CREDENTIAL_ID}" == 'SIT_CREDENTIAL_ID') {

		v_anypointCredentialID = 'TCHINTEGR'

	} else if("${params.ANYPOINT_CREDENTIAL_ID}" == 'PROD_CREDENTIAL_ID') {

		v_anypointCredentialID= 'PCHINTEGR'
	}

	echo "Application name after renaming: ${v_applicationName}"
	echo "Mule Env is : ${v_muleEvn}"
	echo "Encrypt Key : ${v_encryptKey}"
	def v_output=""

/*

										def v_errorMessage = 'Update failed'
				def outLog = new File("${v_jenkinsLogPath}")
				def lines  = outLog.readLines()

				if(lines.any(v_errorMessage)) {
				  echo "BUILD FAILED"
				} else {
				  echo "BUILD SUCCESS"
				}

*/

	if(APP_EXISTS == 'YES') {
		try{
				withCredentials([usernamePassword(credentialsId: "${v_anypointCredentialID}",passwordVariable: 'password',usernameVariable: 'username')]) {
					sh """
						export ANYPOINT_USERNAME=${username}
						export ANYPOINT_PASSWORD=${password}
						echo "**************Password is : ${password}"
						export ANYPOINT_ORG="${v_anypointOrganization}"
						export ANYPOINT_ENV="${v_AnypointEnvironment}"
						anypoint-cli runtime-mgr cloudhub-application modify "${v_applicationName}" \"${v_downloadFilePath}\" --property mule:env="${v_muleEvn}" --property encrypt:key="${v_encryptKey}" --workerSize "${v_cores}" --workers "${v_workers}" --runtime "${v_muleRuntimeEnvironment}"


					"""
				}

			}catch(error){
				throw(error)
				echo "###### RE-DEPLOYMENT IS FAILED ######"
				SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
			}
	} else {
			try {
					withCredentials([usernamePassword(credentialsId: "${v_anypointCredentialID}",passwordVariable: 'password',usernameVariable: 'username')]) {

						sh """
							export ANYPOINT_USERNAME=${username}
							export ANYPOINT_PASSWORD=${password}
							export ANYPOINT_ORG="${v_anypointOrganization}"
							export ANYPOINT_ENV="${v_AnypointEnvironment}"
							anypoint-cli runtime-mgr cloudhub-application deploy "${v_applicationName}" \"${v_downloadFilePath}\" --property mule:env="${v_muleEvn}" --property encrypt:key="${v_encryptKey}" --workerSize "${v_cores}" --workers "${v_workers}" --runtime "${v_muleRuntimeEnvironment}"


						"""
					}
				}catch(error){
					throw(error)
					echo "###### FRESH DEPLOYMENT IS FAILED ######"
					SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
				}
	}
}
/*
This function returns the repository details
*/
def UDF_GetGitRepoName(){
	try{
	def tokens = "${env.JOB_NAME}".tokenize('/')
    String repo = tokens[tokens.size()-2]

    //branch = tokens[tokens.size()-1]
    //org = tokens[tokens.size()-3]

	return repo
	}catch(error)
	{
		throw(error)
		SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
	}
}

/*
This function returns the POM Data
*/
def UDF_GetPOMData(udfp_PomName, udfp_PropertyName){
	try{
	def resultVal = ""
	def pomFile = readFile(udfp_PomName)
	def pom = new XmlParser().parseText(pomFile)
	def gavMap = [:]
	resultVal =  pom[udfp_PropertyName].text().trim()
	return resultVal
	}catch(error)
	{
		throw(error)
		SendEmail("michal.giela@oasis-warehouse.com","Jenkins@oasis-stores.com","Failed")
	}
}

/*
This function Sends Email
*/
def SendEmail(udfp_ToAddress, udfp_FromAddress, udfp_Status)
{
	try{
	   String body = ""

	   if(udfp_Status == "success")
	   {
		   body= "SUCCESS"
	   }
	   else
	   {
		   body= "FAILED"
	   }

		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: "michal.giela@oasis-warehouse.com",
			 replyTo: "webmaster@oasis-stores.com",
				from: "Jenkins@oasis-stores.com"

	}catch(error)
	{
		throw(error)
	}
}
