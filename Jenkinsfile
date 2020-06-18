node {
properties([parameters([choice(choices: 'blue\ngreen', description: 'Select branch to build', name: 'Branch')])])
def servicePrincipalId = 'Jenkins_Kubernetes_Account'
def resourceGroup = 'RG-ZAN-Dev'
def aks = 'RG-ZAN-Dev-Jenkins-Cluster'

def cosmosResourceGroup = 'RG-ZAN-Dev'
def cosmosDbName = 'jenkinsdemocosmosdb'
def dbName = 'todoapp'

def dockerRegistry = 'rgzanjenkinsregistry.azurecr.io'
env.imageNameRedis = "rgzanjenkinsregistry.azurecr.io/redis:latest"
def dockerCredentialId = 'RGZANJenkinsRegistry' 

echo "Pulling changes from the branch ${params.Branch}"
 stage('Scm Checkout'){
  
  git url : 'https://github.com/Shaqfoo2260/azure-voting-app-redis', branch: "${params.Branch}"
  }
 stage('Docker Image Build'){
            echo "${dockerCredentialId}"
        echo "${dockerRegistry}"
        imageName="azure-voting-app-redis-${params.Branch}"
        echo "${imageName}"
	 env.IMAGE_TAG = "${dockerRegistry}/${imageName}:${BUILD_NUMBER}"
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
              az aks get-credentials --resource-group "${resourceGroup}" --name "${aks}" --admin --file kubeconfig  --overwrite-existing
              az logout
	            """
        }	

        // clean the existing environment
    		//sh """
		//kubectl --kubeconfig=kubeconfig delete deployment azure-vote-back-green-$BUILD_NUMBER
		//"""
	}
	stage('Deploy') {
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
}
