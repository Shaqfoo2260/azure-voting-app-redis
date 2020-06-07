  
properties([parameters([choice(choices: 'master\nblue\ngreen', description: 'Select branch to build', name: 'Branch')])])

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
            dir('target') {
                sh """
                    cp -f azure-vote/Dockerfile .
                    docker build -t "${env.IMAGE_TAG}" .
                    docker push "${env.IMAGE_TAG}"
                """
            }
       }
     }
}
