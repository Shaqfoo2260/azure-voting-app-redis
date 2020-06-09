node {
properties([parameters([choice(choices: 'blue\ngreen', description: 'Select branch to build', name: 'Branch')])])
def servicePrincipalId = 'Jenkins_Kubernetes_Account'
def resourceGroup = 'RG-ZAN-Dev'
def aks = 'RG-ZAN-Dev-Jenkins-Cluster'

def cosmosResourceGroup = 'RG-ZAN-Dev'
def cosmosDbName = 'jenkinsdemocosmosdb'
def dbName = 'todoapp'

def dockerRegistry = 'rgzanjenkinsregistry.azurecr.io'
def imageName = ""
def dockerCredentialId = 'RGZANJenkinsRegistry' 

echo "Pulling changes from the branch ${params.Branch}"
 stage('Scm Checkout'){
  
  git url : 'https://github.com/Shaqfoo2260/azure-voting-app-redis', branch: "${params.Branch}"
  }
 stage('Docker Image Build'){
            echo "${dockerCredentialId}"
        echo "${dockerRegistry}"
        imageName="azure-voting-app-redis:${params.Branch}"
        echo "${imageName}"
        env.IMAGE_TAG = "${dockerRegistry}/${imageName}"
        withDockerRegistry([credentialsId: dockerCredentialId, url: "http://${dockerRegistry}"]) {
			sh """
              cd azure-vote/
               docker images -a
               docker build -t "${env.IMAGE_TAG}" .
               docker push "${env.IMAGE_TAG}"
			 """
               }
      
	}
	//stage('Cache admin credentials') {
        // Connect to cluster and cache admin credentials

        withCredentials([azureServicePrincipal(servicePrincipalId)]) {
           // // fetch the current service configuration
          //  sh """
             az login --service-principal -u "\$AZURE_CLIENT_ID" -p "\$AZURE_CLIENT_SECRET" -t "\$AZURE_TENANT_ID"
              az account set --subscription "\$AZURE_SUBSCRIPTION_ID"
             az aks get-credentials --resource-group "${resourceGroup}" --name "${aks}" --admin --file kubeconfig  --overwrite-existing
              az logout
            //  current_role="\$(kubectl --kubeconfig kubeconfig get services cputdemoapp-service --output json | jq -r .spec.selector.role)"
            //  if [ "\$current_role" = null ]; then
            //      echo "Unable to determine current environment"
             //     exit 1
             // fi
             // echo "\$current_role" >current-environment
            """
        }
	//	 currentEnvironment = readFile('current-environment').trim()
        //echo "${currentEnvironment}"

        // set the build name
	//	echo "***************************  CURRENT: $currentEnvironment     NEW: ${params.Branch} *****************************"
        //currentBuild.displayName = newEnvironment().toUpperCase() + ' ' + imageName
       // echo "${currentBuild.displayName}"
        env.TARGET_ROLE = "${params.Branch}"
		echo "${TARGET_ROLE}"
        // clean the inactive environment
    //  sh """
      //      if [[ -d "/var/lib/jenkins/workspace/CPUTDemoApp@tmp" ]] ; then
        //      kubectl --kubeconfig=kubeconfig delete deployment "azure-vote-back-deployment-\$TARGET_ROLE"
       // 	fi
              
        //     """
        
	}
	stage('Deploy green image') {
        // Apply the deployments to AKS.
        // With enableConfigSubstitution set to true, the variables ${TARGET_ROLE}, ${IMAGE_TAG}, ${KUBERNETES_SECRET_NAME}
        // will be replaced with environment variable values
        acsDeploy azureCredentialsId: servicePrincipalId,
                  resourceGroupName: resourceGroup,
                  containerService: "${aks} | AKS",
                  configFilePaths: "azure-vote-all-in-one-redis.yaml",
                  enableConfigSubstitution: true,
                  secretName: dockerRegistry,
                  containerRegistryCredentials: [[credentialsId: dockerCredentialId, url: "http://${dockerRegistry}"]]
    }
	   def verifyEnvironment = { service ->
        sh """
          endpoint_ip="\$(kubectl --kubeconfig=kubeconfig get services '${service}' --output json | jq -r '.status.loadBalancer.ingress[0].ip')"
          count=0
          while true; do
              count=\$(expr \$count + 1)
              if curl -m 10 "http://\$endpoint_ip"; then
                  break;
              fi
              if [ "\$count" -gt 30 ]; then
                  echo 'Timeout while waiting for the ${service} endpoint to be ready'
                  exit 1
              fi
              echo "${service} endpoint is not ready, wait 10 seconds..."
              sleep 10
          done
        """
    }
	
	   stage('Verify green environment') {
        // verify the green environment is working properly
        verifyEnvironment('cputdemoapp-test-green')
    }
	stage('Post-clean') {
        sh '''
          rm -f kubeconfig
        '''
    }
}
