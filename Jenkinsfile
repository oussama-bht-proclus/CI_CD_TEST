pipeline {
  agent any

  //Configure the following environment variables before executing the Jenkins Job
  environment {
    IntegrationFlowID = "temp_iflow"
    IntegrationPackage = "CICD" //relevant for flows that are uploaded the first time 
    DeployFlow = true //if the flow should only be uploaded, set this to false
    DeploymentCheckRetryCounter = 20 //multiply by 3 to get the maximum deployment time
    CPIHost = "d4854fbatrial.it-cpitrial05.cfapps.us10-001.hana.ondemand.com"
    CPIOAuthHost = "d4854fbatrial.it-cpitrial05-rt.cfapps.us10-001.hana.ondemand.com"
    CPIOAuthCredentials = "${env.CPI_OAUTH_CREDS_INTEGRATION}"
    GITRepositoryURL = "github.com/oussama-bht-proclus/CI_CD_TEST"
    GITCredentials = "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"
    GITBranch = "main"
    GITFolder = "IntegrationContent/IntegrationArtefacts"
  }

  stages {
    stage('Get Iflow Artifact from Git, upload to CPI Designtime and optionally Deploy') {
      steps {
		    //empty the workspace
        deleteDir()
		
        script {
		      //checkout the folder from Git
          checkout([
            $class: 'GitSCM',
            branches: [[name: env.GITBranch]],
            doGenerateSubmoduleConfigurations: false,
            extensions: [
              [$class: 'RelativeTargetDirectory', relativeTargetDir: "."],
              [$class: 'SparseCheckoutPaths', sparseCheckoutPaths: [
                [$class: 'SparseCheckoutPath', path: env.GITFolder]
              ]]
            ],
            submoduleCfg: [],
            userRemoteConfigs: [
              [
                credentialsId: env.GITCredentials,
                url: 'https://' + env.GITRepositoryURL
              ]
            ]
          ])

		      //zip the flow content 
          def folder = env.GITFolder + '/' + env.IntegrationFlowID + '/';
          def filePath = env.IntegrationFlowID + ".zip";

          zip dir: folder, glob: '', zipFile: filePath;

          //get token
		println("Requesting token from Cloud Integration tenant");
		println("test print 0")
	  
	          /*def getTokenResp = httpRequest acceptType: 'APPLICATION_JSON',
	            authentication: 'OAUTH_TEST_API',
	            contentType: 'APPLICATION_JSON',
	            httpMode: 'POST',
	            responseHandle: 'LEAVE_OPEN',
	            timeout: 50,
		    consoleLogResponseBody : true,
	            url: 'https://d4854fbatrial.authentication.us10.hana.ondemand.com/oauth/token?grant_type=client_credentials';*/
	  def getTokenResp = httpRequest acceptType: 'APPLICATION_JSON',
	           // authentication: 'OAUTH_TEST_API',
	            contentType: 'APPLICATION_JSON',
	            httpMode: 'POST',
	            responseHandle: 'LEAVE_OPEN',
	            timeout: 50,
		    customHeaders:[[name:'Authorization', value:"Basic c2ItM2JjNGRhMGMtN2E0YS00ZTc0LWE0MDEtNDc3NzY1NTExYzM1IWIyNzkwNDR8aXQhYjI2NjU1OmQwZDRjZjFiLWI5MTUtNDI5Ny1iOWJjLTEzYTI0ZWQwMmFhNyRfeUNDSi1tdE5lOWE3V041aWRKSkU1T216T2JuRFdoM3AxS0ZEb01VaWd3PQ=="]],
		    consoleLogResponseBody : true,
	            url: 'https://d4854fbatrial.authentication.us10.hana.ondemand.com/oauth/token?grant_type=client_credentials';
	 
	  println(" your response object is :"+getTokenResp.getContent())
          //def jsonObjToken = readJSON text: getTokenResp.content
	  def jsonObjToken = new groovy.json.JsonSlurper().parseText(getTokenResp.getContent())
          def token = "Bearer " + jsonObjToken.access_token
	  println("auth token :" + token)

	  getTokenResp.close()
	  //check if the flow already exists on the tenant

	try {
          def checkResp = httpRequest acceptType: 'APPLICATION_JSON',
            httpMode: 'GET',
            responseHandle: 'LEAVE_OPEN',
            //validResponseCodes: '200,201,202,404',
            timeout: 50,
	    contentType : 'APPLICATION_JSON',
	    customHeaders:[[name:'Authorization', value:"Bearer eyJhbGciOiJSUzI1NiIsImprdSI6Imh0dHBzOi8vZDQ4NTRmYmF0cmlhbC5hdXRoZW50aWNhdGlvbi51czEwLmhhbmEub25kZW1hbmQuY29tL3Rva2VuX2tleXMiLCJraWQiOiJkZWZhdWx0LWp3dC1rZXktZWZlMmY1YjA0ZiIsInR5cCI6IkpXVCIsImppZCI6ICIxVjdMR0Y2SmF6L2dlR3dCVlRROUJCQ1hZTDdRdWtveXhrTzJGVWlRdG9rPSJ9.eyJqdGkiOiI5ZjMzZjE1NzYyNWU0Nzc4YTFkOGFkMTE2NDE2NWM4YiIsImV4dF9hdHRyIjp7ImVuaGFuY2VyIjoiWFNVQUEiLCJzdWJhY2NvdW50aWQiOiIyMzVjNThjZC04ZTUxLTRjZGItOGE1Ni0wYTM4YjNiMjk2OWYiLCJ6ZG4iOiJkNDg1NGZiYXRyaWFsIiwic2VydmljZWluc3RhbmNlaWQiOiIzYmM0ZGEwYy03YTRhLTRlNzQtYTQwMS00Nzc3NjU1MTFjMzUifSwic3ViIjoic2ItM2JjNGRhMGMtN2E0YS00ZTc0LWE0MDEtNDc3NzY1NTExYzM1IWIyNzkwNDR8aXQhYjI2NjU1IiwiYXV0aG9yaXRpZXMiOlsiaXQhYjI2NjU1LkRhdGFBcmNoaXZpbmcuQWN0aXZhdGUiLCJpdCFiMjY2NTUuTWVzc2FnZVVzYWdlRGFzaGJvYXJkLlJlYWQiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ1dvcmtzcGFjZS5QdWJsaXNoIiwiaXQhYjI2NjU1LkdvdmVybmFuY2UuR292ZXJuYW5jZUNvbW1lbnRzV3JpdGUiLCJpdCFiMjY2NTUuV2ViVG9vbGluZy5JbnRlZ3JhdGlvbkZsb3dDb25maWd1cmUiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ1dvcmtzcGFjZS5SZWFkIiwiaXQhYjI2NjU1LldlYlRvb2xpbmcuRFNPREludGVncmF0aW9uIiwiaXQhYjI2NjU1LldvcmtzcGFjZUFydGlmYWN0TG9ja3MuRGVsZXRlIiwiaXQhYjI2NjU1LkNvbXBhbnlTZW5zaXRpdmVEYXRhLlJlYWQiLCJpdCFiMjY2NTUuQWNjZXNzUG9saWNpZXNBcnRpZmFjdHMuQWNjZXNzQWxsIiwiaXQhYjI2NjU1LkNvZGVsaXN0LlJlYWQiLCJpdCFiMjY2NTUuQ29tcGFueVByb2ZpbGUuUmVhZCIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lclNlbnNpdGl2ZURhdGEuUmVhZCIsIml0IWIyNjY1NS5XZWJUb29saW5nV29ya3NwYWNlLldyaXRlIiwiaXQhYjI2NjU1LkVTQkRhdGFTdG9yZS5kZWxldGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuUmVzdGFydENvbXBvbmVudCIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUuQWN0aXZhdGUiLCJpdCFiMjY2NTUuRVNCRGF0YVN0b3JlLkNvbmZpZyIsIml0IWIyNjY1NS5EZWZhdWx0IiwiaXQhYjI2NjU1LlRlbmFudFBhcnRuZXJEaXJlY3RvcnkucmVhZCIsInVhYS5yZXNvdXJjZSIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lclByb2ZpbGUuV3JpdGUiLCJpdCFiMjY2NTUuTWVzc2FnZVByb2Nlc3NpbmdMb2Nrcy5EZWxldGUiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsQ29tcG9uZW50LlJlYWQiLCJpdCFiMjY2NTUuQ29tcGFueVNlbnNpdGl2ZURhdGEuV3JpdGUiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJQcm9maWxlLlJlYWQiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJBZ3JlZW1lbnQuUHVibGlzaCIsIml0IWIyNjY1NS5lc2JtZXNzYWdlc3RvcmFnZS5yZWFkIiwiaXQhYjI2NjU1Lk1lc3NhZ2VQcm9jZXNzaW5nTG9ja3MuUmVhZCIsIml0IWIyNjY1NS5Db25maWd1cmF0aW9uU2VydmljZS5SdW50aW1lQnVzaW5lc3NQYXJhbWV0ZXJXcml0ZSIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lckFncmVlbWVudC5SZWFkIiwiaXQhYjI2NjU1LkFncmVlbWVudFRlbXBsYXRlLlJlYWQiLCJpdCFiMjY2NTUuRVNCRGF0YVN0b3JlLnJlYWQiLCJpdCFiMjY2NTUuTWVzc2FnZVByb2Nlc3NpbmdMb2cuU3RhdHVzQ2hhbmdlIiwiaXQhYjI2NjU1LlRyYWRpbmdQYXJ0bmVyU2Vuc2l0aXZlRGF0YS5Xcml0ZSIsIml0IWIyNjY1NS5XZWJUb29saW5nU2V0dGluZ3NQcm9kdWN0UHJvZmlsZXMuc2F2ZXRlbmFudGNvbmZpZ3VyYXRpb24iLCJpdCFiMjY2NTUuR292ZXJuYW5jZS5Hb3Zlcm5hbmNlQ29tbWVudHNSZWFkIiwiaXQhYjI2NjU1LkludGVncmF0aW9uQ2VsbFJ1bnRpbWVQYXJhbWV0ZXIuUmVhZCIsIml0IWIyNjY1NS5BY2Nlc3NQb2xpY2llcy5SZWFkIiwiaXQhYjI2NjU1LkludGVncmF0aW9uT3BlcmF0aW9uU2VydmVyLnJlYWQiLCJpdCFiMjY2NTUuUm9sZXMuV3JpdGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIucmVhZCIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lckFncmVlbWVudC5Xcml0ZSIsIml0IWIyNjY1NS5QSVByb3Zpc2lvbmluZy53cml0ZSIsIml0IWIyNjY1NS5HZW5lcmF0aW9uQW5kQnVpbGQuZ2VuZXJhdGlvbmFuZGJ1aWxkY29udGVudCIsIml0IWIyNjY1NS5Db21wYW55UHJvZmlsZS5Xcml0ZSIsIml0IWIyNjY1NS5XZWJUb29saW5nQ2F0YWxvZy5Eb3dubG9hZCIsIml0IWIyNjY1NS5JbnRlZ3JhdGlvbkNlbGxPcGVyYXRpb25Db2NrcGl0LlJlYWQiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuZGVwbG95Y3JlZGVudGlhbHMiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ0NhdGFsb2cuRGV0YWlsc1JlYWQiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuZGVwbG95c2VjdXJpdHljb250ZW50IiwiaXQhYjI2NjU1LkludGVncmF0aW9uT3BlcmF0aW9uU2VydmVyLm1vZGlmeW9wZXJhdGlvbnNqb2JzIiwiaXQhYjI2NjU1Lk5vZGVNYW5hZ2VyLmRlcGxveWNvbnRlbnQiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsUnVudGltZVBhcmFtZXRlci5Xcml0ZSIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUucmV0cnkiLCJpdCFiMjY2NTUuRXh0ZXJuYWxMb2dnaW5nLkFjdGl2YXRlIiwiaXQhYjI2NjU1LlJvbGVzLlJlYWQiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ0NhdGFsb2cuT3ZlcnZpZXdSZWFkIiwiaXQhYjI2NjU1LkRhdGFBcmNoaXZpbmcuUmVhZCIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUucmVhZFBheWxvYWQiLCJpdCFiMjY2NTUuQWdyZWVtZW50VGVtcGxhdGUuV3JpdGUiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsQ29tcG9uZW50LlJlc3RhcnQiLCJpdCFiMjY2NTUuQWNjZXNzUG9saWNpZXMuV3JpdGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIucmVhZGNyZWRlbnRpYWxzIiwiaXQhYjI2NjU1LldvcmtzcGFjZUFydGlmYWN0TG9ja3MuUmVhZCIsIml0IWIyNjY1NS5Xb3Jrc3BhY2VEZXNpZ25HdWlkZWxpbmVzLkNvbmZpZ3VyZSIsIml0IWIyNjY1NS5UZW5hbnRQYXJ0bmVyRGlyZWN0b3J5LndyaXRlIiwiaXQhYjI2NjU1LkV4dGVybmFsTG9nZ2luZ0FjdGl2YXRpb24uUmVhZCIsIml0IWIyNjY1NS5XZWJUb29saW5nQ2F0YWxvZy5DcmVhdGUiLCJpdCFiMjY2NTUuRVNCTWVzc2FnZVN0b3JhZ2UuRGVsZXRlIiwiaXQhYjI2NjU1Lk5vZGVNYW5hZ2VyLnJlYWRzZWN1cml0eWNvbnRlbnQiXSwic2NvcGUiOlsiaXQhYjI2NjU1LkRhdGFBcmNoaXZpbmcuQWN0aXZhdGUiLCJpdCFiMjY2NTUuTWVzc2FnZVVzYWdlRGFzaGJvYXJkLlJlYWQiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ1dvcmtzcGFjZS5QdWJsaXNoIiwiaXQhYjI2NjU1LkdvdmVybmFuY2UuR292ZXJuYW5jZUNvbW1lbnRzV3JpdGUiLCJpdCFiMjY2NTUuV2ViVG9vbGluZy5JbnRlZ3JhdGlvbkZsb3dDb25maWd1cmUiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ1dvcmtzcGFjZS5SZWFkIiwiaXQhYjI2NjU1LldlYlRvb2xpbmcuRFNPREludGVncmF0aW9uIiwiaXQhYjI2NjU1LldvcmtzcGFjZUFydGlmYWN0TG9ja3MuRGVsZXRlIiwiaXQhYjI2NjU1LkNvbXBhbnlTZW5zaXRpdmVEYXRhLlJlYWQiLCJpdCFiMjY2NTUuQWNjZXNzUG9saWNpZXNBcnRpZmFjdHMuQWNjZXNzQWxsIiwiaXQhYjI2NjU1LkNvZGVsaXN0LlJlYWQiLCJpdCFiMjY2NTUuQ29tcGFueVByb2ZpbGUuUmVhZCIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lclNlbnNpdGl2ZURhdGEuUmVhZCIsIml0IWIyNjY1NS5XZWJUb29saW5nV29ya3NwYWNlLldyaXRlIiwiaXQhYjI2NjU1LkVTQkRhdGFTdG9yZS5kZWxldGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuUmVzdGFydENvbXBvbmVudCIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUuQWN0aXZhdGUiLCJpdCFiMjY2NTUuRVNCRGF0YVN0b3JlLkNvbmZpZyIsIml0IWIyNjY1NS5EZWZhdWx0IiwiaXQhYjI2NjU1LlRlbmFudFBhcnRuZXJEaXJlY3RvcnkucmVhZCIsInVhYS5yZXNvdXJjZSIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lclByb2ZpbGUuV3JpdGUiLCJpdCFiMjY2NTUuTWVzc2FnZVByb2Nlc3NpbmdMb2Nrcy5EZWxldGUiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsQ29tcG9uZW50LlJlYWQiLCJpdCFiMjY2NTUuQ29tcGFueVNlbnNpdGl2ZURhdGEuV3JpdGUiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJQcm9maWxlLlJlYWQiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJBZ3JlZW1lbnQuUHVibGlzaCIsIml0IWIyNjY1NS5lc2JtZXNzYWdlc3RvcmFnZS5yZWFkIiwiaXQhYjI2NjU1Lk1lc3NhZ2VQcm9jZXNzaW5nTG9ja3MuUmVhZCIsIml0IWIyNjY1NS5Db25maWd1cmF0aW9uU2VydmljZS5SdW50aW1lQnVzaW5lc3NQYXJhbWV0ZXJXcml0ZSIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lckFncmVlbWVudC5SZWFkIiwiaXQhYjI2NjU1LkFncmVlbWVudFRlbXBsYXRlLlJlYWQiLCJpdCFiMjY2NTUuRVNCRGF0YVN0b3JlLnJlYWQiLCJpdCFiMjY2NTUuTWVzc2FnZVByb2Nlc3NpbmdMb2cuU3RhdHVzQ2hhbmdlIiwiaXQhYjI2NjU1LlRyYWRpbmdQYXJ0bmVyU2Vuc2l0aXZlRGF0YS5Xcml0ZSIsIml0IWIyNjY1NS5XZWJUb29saW5nU2V0dGluZ3NQcm9kdWN0UHJvZmlsZXMuc2F2ZXRlbmFudGNvbmZpZ3VyYXRpb24iLCJpdCFiMjY2NTUuR292ZXJuYW5jZS5Hb3Zlcm5hbmNlQ29tbWVudHNSZWFkIiwiaXQhYjI2NjU1LkludGVncmF0aW9uQ2VsbFJ1bnRpbWVQYXJhbWV0ZXIuUmVhZCIsIml0IWIyNjY1NS5BY2Nlc3NQb2xpY2llcy5SZWFkIiwiaXQhYjI2NjU1LkludGVncmF0aW9uT3BlcmF0aW9uU2VydmVyLnJlYWQiLCJpdCFiMjY2NTUuUm9sZXMuV3JpdGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIucmVhZCIsIml0IWIyNjY1NS5UcmFkaW5nUGFydG5lckFncmVlbWVudC5Xcml0ZSIsIml0IWIyNjY1NS5QSVByb3Zpc2lvbmluZy53cml0ZSIsIml0IWIyNjY1NS5HZW5lcmF0aW9uQW5kQnVpbGQuZ2VuZXJhdGlvbmFuZGJ1aWxkY29udGVudCIsIml0IWIyNjY1NS5Db21wYW55UHJvZmlsZS5Xcml0ZSIsIml0IWIyNjY1NS5XZWJUb29saW5nQ2F0YWxvZy5Eb3dubG9hZCIsIml0IWIyNjY1NS5JbnRlZ3JhdGlvbkNlbGxPcGVyYXRpb25Db2NrcGl0LlJlYWQiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuZGVwbG95Y3JlZGVudGlhbHMiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ0NhdGFsb2cuRGV0YWlsc1JlYWQiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIuZGVwbG95c2VjdXJpdHljb250ZW50IiwiaXQhYjI2NjU1LkludGVncmF0aW9uT3BlcmF0aW9uU2VydmVyLm1vZGlmeW9wZXJhdGlvbnNqb2JzIiwiaXQhYjI2NjU1Lk5vZGVNYW5hZ2VyLmRlcGxveWNvbnRlbnQiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsUnVudGltZVBhcmFtZXRlci5Xcml0ZSIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUucmV0cnkiLCJpdCFiMjY2NTUuRXh0ZXJuYWxMb2dnaW5nLkFjdGl2YXRlIiwiaXQhYjI2NjU1LlJvbGVzLlJlYWQiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ0NhdGFsb2cuT3ZlcnZpZXdSZWFkIiwiaXQhYjI2NjU1LkRhdGFBcmNoaXZpbmcuUmVhZCIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUucmVhZFBheWxvYWQiLCJpdCFiMjY2NTUuQWdyZWVtZW50VGVtcGxhdGUuV3JpdGUiLCJpdCFiMjY2NTUuSW50ZWdyYXRpb25DZWxsQ29tcG9uZW50LlJlc3RhcnQiLCJpdCFiMjY2NTUuQWNjZXNzUG9saWNpZXMuV3JpdGUiLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIucmVhZGNyZWRlbnRpYWxzIiwiaXQhYjI2NjU1LldvcmtzcGFjZUFydGlmYWN0TG9ja3MuUmVhZCIsIml0IWIyNjY1NS5Xb3Jrc3BhY2VEZXNpZ25HdWlkZWxpbmVzLkNvbmZpZ3VyZSIsIml0IWIyNjY1NS5UZW5hbnRQYXJ0bmVyRGlyZWN0b3J5LndyaXRlIiwiaXQhYjI2NjU1LkV4dGVybmFsTG9nZ2luZ0FjdGl2YXRpb24uUmVhZCIsIml0IWIyNjY1NS5XZWJUb29saW5nQ2F0YWxvZy5DcmVhdGUiLCJpdCFiMjY2NTUuRVNCTWVzc2FnZVN0b3JhZ2UuRGVsZXRlIiwiaXQhYjI2NjU1Lk5vZGVNYW5hZ2VyLnJlYWRzZWN1cml0eWNvbnRlbnQiXSwiY2xpZW50X2lkIjoic2ItM2JjNGRhMGMtN2E0YS00ZTc0LWE0MDEtNDc3NzY1NTExYzM1IWIyNzkwNDR8aXQhYjI2NjU1IiwiY2lkIjoic2ItM2JjNGRhMGMtN2E0YS00ZTc0LWE0MDEtNDc3NzY1NTExYzM1IWIyNzkwNDR8aXQhYjI2NjU1IiwiYXpwIjoic2ItM2JjNGRhMGMtN2E0YS00ZTc0LWE0MDEtNDc3NzY1NTExYzM1IWIyNzkwNDR8aXQhYjI2NjU1IiwiZ3JhbnRfdHlwZSI6ImNsaWVudF9jcmVkZW50aWFscyIsInJldl9zaWciOiI1NjA2ZDI1IiwiaWF0IjoxNzE1NjcwMzcxLCJleHAiOjE3MTU3MTM1NzEsImlzcyI6Imh0dHBzOi8vZDQ4NTRmYmF0cmlhbC5hdXRoZW50aWNhdGlvbi51czEwLmhhbmEub25kZW1hbmQuY29tL29hdXRoL3Rva2VuIiwiemlkIjoiMjM1YzU4Y2QtOGU1MS00Y2RiLThhNTYtMGEzOGIzYjI5NjlmIiwiYXVkIjpbIml0IWIyNjY1NS5BY2Nlc3NQb2xpY2llcyIsIml0IWIyNjY1NS5NZXNzYWdlUHJvY2Vzc2luZ0xvY2tzIiwiaXQhYjI2NjU1LldlYlRvb2xpbmdTZXR0aW5nc1Byb2R1Y3RQcm9maWxlcyIsIml0IWIyNjY1NS5JbnRlZ3JhdGlvbkNlbGxDb21wb25lbnQiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJTZW5zaXRpdmVEYXRhIiwiaXQhYjI2NjU1Lk1lc3NhZ2VVc2FnZURhc2hib2FyZCIsIml0IWIyNjY1NS5Hb3Zlcm5hbmNlIiwiaXQhYjI2NjU1LlJvbGVzIiwiaXQhYjI2NjU1LkludGVncmF0aW9uQ2VsbFJ1bnRpbWVQYXJhbWV0ZXIiLCJpdCFiMjY2NTUuV29ya3NwYWNlQXJ0aWZhY3RMb2NrcyIsIml0IWIyNjY1NS5XZWJUb29saW5nQ2F0YWxvZyIsIml0IWIyNjY1NS5BY2Nlc3NQb2xpY2llc0FydGlmYWN0cyIsInVhYSIsIml0IWIyNjY1NS5Xb3Jrc3BhY2VEZXNpZ25HdWlkZWxpbmVzIiwiaXQhYjI2NjU1LkdlbmVyYXRpb25BbmRCdWlsZCIsIml0IWIyNjY1NS5Db2RlbGlzdCIsIml0IWIyNjY1NS5JbnRlZ3JhdGlvbk9wZXJhdGlvblNlcnZlciIsIml0IWIyNjY1NSIsIml0IWIyNjY1NS5QSVByb3Zpc2lvbmluZyIsIml0IWIyNjY1NS5FU0JEYXRhU3RvcmUiLCJpdCFiMjY2NTUuRGF0YUFyY2hpdmluZyIsIml0IWIyNjY1NS5UZW5hbnRQYXJ0bmVyRGlyZWN0b3J5IiwiaXQhYjI2NjU1LkVTQk1lc3NhZ2VTdG9yYWdlIiwiaXQhYjI2NjU1LkV4dGVybmFsTG9nZ2luZyIsIml0IWIyNjY1NS5lc2JtZXNzYWdlc3RvcmFnZSIsIml0IWIyNjY1NS5Db25maWd1cmF0aW9uU2VydmljZSIsIml0IWIyNjY1NS5JbnRlZ3JhdGlvbkNlbGxPcGVyYXRpb25Db2NrcGl0IiwiaXQhYjI2NjU1LkFncmVlbWVudFRlbXBsYXRlIiwiaXQhYjI2NjU1LkV4dGVybmFsTG9nZ2luZ0FjdGl2YXRpb24iLCJpdCFiMjY2NTUuTm9kZU1hbmFnZXIiLCJpdCFiMjY2NTUuVHJhZGluZ1BhcnRuZXJBZ3JlZW1lbnQiLCJpdCFiMjY2NTUuQ29tcGFueVNlbnNpdGl2ZURhdGEiLCJzYi0zYmM0ZGEwYy03YTRhLTRlNzQtYTQwMS00Nzc3NjU1MTFjMzUhYjI3OTA0NHxpdCFiMjY2NTUiLCJpdCFiMjY2NTUuQ29tcGFueVByb2ZpbGUiLCJpdCFiMjY2NTUuV2ViVG9vbGluZ1dvcmtzcGFjZSIsIml0IWIyNjY1NS5XZWJUb29saW5nIiwiaXQhYjI2NjU1LlRyYWRpbmdQYXJ0bmVyUHJvZmlsZSIsIml0IWIyNjY1NS5NZXNzYWdlUHJvY2Vzc2luZ0xvZyJdfQ.oUcDYFen3nGi92bIF9KCZ7Q3n_pzKqPty5gm4gcC64u2NQ3OSqSH6ZZb8l1hYIZ0jOXlKTNuNZAN_1fLI0SonR3X0Xq98s4dB8jkDJPBsacHV4AndxjMIX7MZ5_uu69dG5sH4Hx-TE5X41Xl_UtIOE8K4OBEDhBPMRB-TnQUlZqYSYfxleqfOMR4ESHJ_eGms0uOImcXjV03DOI1Yhh5r66p4apMYH-ZWiPQqUiBwMrzzC7jq0EL3bDj2Ulk7R-FX6dfFa8r4NopZ5qDs-d49jYq55Bt-w-hmWWGa_IHkZWu6Pp33XUiE3eViuabT4Hv1ld-cYA2yeaXl6RjO3EFcw"]],
	    consoleLogResponseBody : false,
            url: 'https://d4854fbatrial.it-cpitrial05.cfapps.us10-001.hana.ondemand.com/api/v1/IntegrationDesigntimeArtifacts(Id=\'' + env.IntegrationFlowID + '\',Version=\'active\')';
	}catch(Exception err){
		println("error somewhere")
		println("Error occurred while making HTTP request:")
		    //println("Error message: ${err.message}")
		    //println("Stack trace:")
		    //err.printStackTrace()
		     println("Error class: ${err.getClass().getName()}")
    		     println("Error object: $err")
	}

	  println("test print 3")
	  def temp = new groovy.json.JsonSlurper().parseText(checkResp.getContent())
	  println(temp)
          def filecontent = readFile encoding: 'Base64', file: filePath;
          if (checkResp.status == 404) {
            //Upload integration flow via POST
			      println("Flow does not yet exist on configured tenant.");
            //prepare upload payload
            def postPayload = '{ \"Name\": \"flowName\", \"Id": "flowId\", \"PackageId\": \"packageId\", \"ArtifactContent\":\"flowContent\"}';

            postPayload = postPayload.replace('flowName', env.IntegrationFlowID);
            postPayload = postPayload.replace('flowId', env.IntegrationFlowID);
            postPayload = postPayload.replace('packageId', env.IntegrationPackage);
            postPayload = postPayload.replace('flowContent', filecontent);

            //upload
			      println("Uploading flow.");
            def postResp = httpRequest acceptType: 'APPLICATION_JSON',
              contentType: 'APPLICATION_JSON',
              customHeaders: [
                [maskValue: false, name: 'Authorization', value: token]
              ],
              httpMode: 'POST',
              requestBody: postPayload,
              url: 'https://' + env.CPIHost + '/api/v1/IntegrationDesigntimeArtifacts'
          } else {
            //Overwrite integration flow via PUT
			      println("Flow already exists on configured tenant. Update will be performed.");
            //prepare upload payload
            def putPayload = '{ \"Name\": \"flowName\", \"ArtifactContent\": \"iflowContent\"}';
            putPayload = putPayload.replace('flowName', env.IntegrationFlowID);
            putPayload = putPayload.replace('iflowContent', filecontent);

            //upload
			      println("Uploading flow.");
            def putResp = httpRequest acceptType: 'APPLICATION_JSON',
              contentType: 'APPLICATION_JSON',
              customHeaders: [
                [maskValue: false, name: 'Authorization', value: token]
              ],
              httpMode: 'PUT',
              requestBody: putPayload,
              url: 'https://' + env.CPIHost + '/api/v1/IntegrationDesigntimeArtifacts(Id=\'' + env.IntegrationFlowID + '\',Version=\'active\')';
          }
          println("Upload successful");
          checkResp.close();

          if (env.DeployFlow.equalsIgnoreCase("true")) {
            //deploy integration flow
            println("Deploying integration flow");
            def deployResp = httpRequest httpMode: 'POST',
              customHeaders: [
                [maskValue: false, name: 'Authorization', value: token]
              ],
              ignoreSslErrors: true,
              timeout: 30,
              url: 'https://' + env.CPIHost + '/api/v1/DeployIntegrationDesigntimeArtifact?Id=\'' + env.IntegrationFlowID + '\'&Version=\'active\'';

            Integer counter = 0;
            def deploymentStatus;
            def continueLoop = true;
            println("Deployment successful triggered. Checking status.");
			      //performing the loop until we get a final deployment status.
            while (counter < env.DeploymentCheckRetryCounter.toInteger() & continueLoop == true) {
              Thread.sleep(3000);
              counter = counter + 1;
              def statusResp = httpRequest acceptType: 'APPLICATION_JSON',
                customHeaders: [
                  [maskValue: false, name: 'Authorization', value: token]
                ],
                httpMode: 'GET',
                responseHandle: 'LEAVE_OPEN',
                timeout: 30,
                url: 'https://' + env.CPIHost + '/api/v1/IntegrationRuntimeArtifacts(\'' + env.IntegrationFlowID + '\')';
              def jsonObj = readJSON text: statusResp.content;
              deploymentStatus = jsonObj.d.Status;

              println("Deployment status: " + deploymentStatus);
              if (deploymentStatus.equalsIgnoreCase("Error")) {
                //get error details
                def deploymentErrorResp = httpRequest acceptType: 'APPLICATION_JSON',
                  customHeaders: [
                    [maskValue: false, name: 'Authorization', value: token]
                  ],
                  httpMode: 'GET',
                  responseHandle: 'LEAVE_OPEN',
                  timeout: 30,
                  url: 'https://' + env.CPIHost + '/api/v1/IntegrationRuntimeArtifacts(\'' + env.IntegrationFlowID + '\')' + '/ErrorInformation/$value';
                def jsonErrObj = readJSON text: deploymentErrorResp.content
                def deployErrorInfo = jsonErrObj.parameter;
                println("Error Details: " + deployErrorInfo);
                statusResp.close();
                deploymentErrorResp.close();
				        error("Integration flow not deployed successfully. Ending job now.");
              } else if (deploymentStatus.equalsIgnoreCase("Started")) {
                println("Integration flow deployment was successful")
                statusResp.close();
                continueLoop = false
              } else {
                println("The integration flow is not yet started. Will wait 3s and then check again.")
              }
            }
            if (!deploymentStatus.equalsIgnoreCase("Started")) {
              error("No final deployment status could be reached. Kindly check the tenant for any issue.");
            }
          }
        }
      }
    }
  }
}
