#!groovy

/**
 *
 * Author:    Naresh Manthrabuddi
 * Created:   12.12.2019
 * 
 **/

pipeline {
  agent any
  
	stages {
	    stage('Pipeline gets Started') {		
			steps {
			   echo 'Oasis Project CI/CD Pipeline Started'
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
	def v_muleEnv = ""

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
         choices: '4.1.6\n4.2.0\n4.2.1',
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
            name: 'WORKERTYPE'
       ),
       string(
		   name: 'APPLICATION_NAME', 
		   defaultValue: 'cicd-demo', 
		   description: 'Please enter the application name for CloudHub Deployment?'
	   ),
       string(
		   name: 'ANYPOINT_ORGANIZATION', 
		   defaultValue: 'Warehouse Fashions', 
		   description: 'Please provide CloudHub Anypoint Organization name to deploy?'
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
		bat 'mvn clean package'	
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
		bat 'mvn sonar:sonar'
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
	v_cores = "${params.WORKERTYPE}"
	v_applicationName = "${params.APPLICATION_NAME}"
	v_anypointCredentialID = ""
	v_anypointOrganization = "${params.ANYPOINT_ORGANIZATION}"
	v_anypointEnvironment = "${params.ANYPOINT_ENVIRONMENT}"
	v_muleEvn = ""
	v_encryptKey = ""
	v_artifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
	v_version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
	v_package = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
	v_downloadFilePath = "${env.WORKSPACE}\\target\\${v_artifactId}-${v_version}-${v_package}.jar"	

	if("${params.ENVIRONMENT}" == 'DEV') {

		v_muleEnv = 'dev-'
		v_anypointCredentialID = '2485368e-fd54-44eb-a655-2dbab025daa2'

	} else if("${params.ENVIRONMENT}" == 'SIT') {
		v_muleEnv = 'sit-'
		v_anypointCredentialID= '2485368e-fd54-44eb-a655-2dbab025daa2'

	} else if("${params.ENVIRONMENT}" == 'UAT') {
		v_muleEnv = 'uat-'
		v_anypointCredentialID= '2485368e-fd54-44eb-a655-2dbab025daa2'
	
	} else if("${params.ENVIRONMENT}" == 'PROD') {
		v_muleEnv = ''
		v_anypointCredentialID= '2485368e-fd54-44eb-a655-2dbab025daa2'
	}

	echo "ENVIRONMENT is : ${v_environment}"
	echo "BUILD_MECHANISM is : ${v_buildMechanism}"
	echo "MULE_RUNTIME_VERSION is : ${v_muleRuntimeEnvironment}"
	echo "WORKERS : ${v_workers}"
	echo "WORKERTYPE : ${v_cores}"
	echo "APPLICATION_NAME is : ${v_applicationName}"
	echo "ANYPOINT_ORGANIZATION is : ${v_anypointOrganization}"
	echo "Environment Workspace is : ${env.WORKSPACE}"	
	echo "Download File Path is : ${v_downloadFilePath}"
	echo "ANYPOINT_CREDENTIAL_ID is : ${v_anypointCredentialID}"
	echo "MULE_ENVIRONMENT is : ${v_muleEnv}"

	withCredentials([usernamePassword(credentialsId: "${v_anypointCredentialID}",passwordVariable: 'ANYPOINT_PASSWORD',usernameVariable: 'ANYPOINT_USERNAME')]) {
		bat "mvn deploy -DmuleDeploy -Danypoint.username=${ANYPOINT_USERNAME} -Danypoint.password=${ANYPOINT_PASSWORD} -Denvironment=${v_environment} -DbusinessGroup=\"${v_anypointOrganization}\" -DmuleVersion=${v_muleRuntimeEnvironment} -DapplicationName=${v_muleEnv}${v_applicationName}"
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
		/*
		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: "michal.giela@oasis-warehouse.com",
			 replyTo: "webmaster@oasis-stores.com",
				from: "Jenkins@oasis-stores.com"
		*/
				
	}catch(error)
	{		
		throw(error)
	}
}
