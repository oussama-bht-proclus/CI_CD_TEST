pipeline {
  agent any

  //Configure the following environment variables before executing the Jenkins Job
  environment {
    IntegrationFlowID = "IntegrationFlow1"
    IntegrationPackage = "CICD" //relevant for flows that are uploaded the first time 
    DeployFlow = true //if the flow should only be uploaded, set this to false
    DeploymentCheckRetryCounter = 20 //multiply by 3 to get the maximum deployment time
    CPIHost = "d4854fbatrial.it-cpitrial05.cfapps.us10-001.hana.ondemand.com"
    //CPIOAuthHost = "https://d4854fbatrial.authentication.us10.hana.ondemand.com"
    //CPIOAuthHost = "d4854fbatrial.authentication.us10.hana.ondemand.com"
    CPIOAuthHost = "d4854fbatrial.authentication.us10.hana.ondemand.com"
    CPIOAuthCredentials = "c2ItODJiMzMyZGItNjBiMi00NzFmLTkxNTEtYjVkYWJiNzUwMGMxIWIyNzkwNDR8aXQtcnQtZDQ4NTRmYmF0cmlhbCFiMjY2NTU6NzgxNzY3OGUtMGE4Mi00YTZmLTllYzMtZTk2YTY3YTk3NWZlJFBFNUtZSWp6NVpmRGlnSHE3TUkxWjZ3NDRxNUl6VDFZT3dNbGRXenVzaGs9"
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
		println("test print")
	  /*try {*/
	          def getTokenResp = httpRequest acceptType: 'APPLICATION_JSON',
	            authentication: 'c2ItODJiMzMyZGItNjBiMi00NzFmLTkxNTEtYjVkYWJiNzUwMGMxIWIyNzkwNDR8aXQtcnQtZDQ4NTRmYmF0cmlhbCFiMjY2NTU6NzgxNzY3OGUtMGE4Mi00YTZmLTllYzMtZTk2YTY3YTk3NWZlJFBFNUtZSWp6NVpmRGlnSHE3TUkxWjZ3NDRxNUl6VDFZT3dNbGRXenVzaGs9',
	            contentType: 'APPLICATION_JSON',
	            httpMode: 'POST',
	            responseHandle: 'LEAVE_OPEN',
	            timeout: 50,
	            url: 'https://d4854fbatrial.authentication.us10.hana.ondemand.com/oauth/token?grant_type=client_credentials';
	 /* catch (Exception e) {
	      println("Error occurred while obtaining token: " + e.getMessage());
	      return; // Stop execution if there's an error
	  }*/
	  println("test print 1")
          def jsonObjToken = readJSON text: getTokenResp.content
		println("test print 2")
          def token = "Bearer " + jsonObjToken.access_token
		println("test test");

          //check if the flow already exists on the tenant
          def checkResp = httpRequest acceptType: 'APPLICATION_JSON',
            customHeaders: [
              [maskValue: false, name: 'Authorization', value: token]
            ],
            httpMode: 'GET',
            responseHandle: 'LEAVE_OPEN',
            validResponseCodes: '200,201,202,404',
            timeout: 30,
            url: 'https://' + env.CPIHost + '/api/v1/IntegrationDesigntimeArtifacts(Id=\'' + env.IntegrationFlowID + '\',Version=\'active\')';

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
