  
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

node {
 stage('Scm Checkout'){
  echo "Pulling changes from the branch ${params.Branch}"
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
	stage('Check Env') {
        // check the current active environment to determine the inactive one that will be deployed to

        withCredentials([azureServicePrincipal(servicePrincipalId)]) {
            // fetch the current service configuration
            sh """
              az login --service-principal -u "\$AZURE_CLIENT_ID" -p "\$AZURE_CLIENT_SECRET" -t "\$AZURE_TENANT_ID"
              az account set --subscription "\$AZURE_SUBSCRIPTION_ID"
              az aks get-credentials --resource-group "${resourceGroup}" --name "${aks}" --admin --file kubeconfig
              az logout
              current_role="\$(kubectl --kubeconfig kubeconfig get services todoapp-service --output json | jq -r .spec.selector.role)"
              if [ "\$current_role" = null ]; then
                  echo "Unable to determine current environment"
                  exit 1
              fi
              echo "\$current_role" >current-environment
            """
        }
		 currentEnvironment = readFile('current-environment').trim()
        echo "${currentEnvironment}"

        // set the build name
        echo "***************************  CURRENT: $currentEnvironment     NEW: "${params.Branch}" *****************************"
        //currentBuild.displayName = newEnvironment().toUpperCase() + ' ' + imageName
       // echo "${currentBuild.displayName}"
        env.TARGET_ROLE = ${params.Branch}
		echo "${TARGET_ROLE}"
        // clean the inactive environment
     //  sh """
      //      if [[ -d "/var/lib/jenkins/workspace/Todoapp@tmp/todoapp-deployment" ]] ; then
      //        kubectl --kubeconfig=kubeconfig delete deployment "todoapp-deployment-\$TARGET_ROLE"
      //  	fi
              
        //      """
        
	}
}
