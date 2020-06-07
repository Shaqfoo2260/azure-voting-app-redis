properties([parameters([choice(choices: 'master\nblue\ngreen', description: 'Select branch to build', name: 'Branch')])])
 
node {
 stage('Scm Checkout'){
  echo "Pulling changes from the branch ${params.Branch}"
  git url : 'https://github.com/Shaqfoo2260/azure-voting-app-redis', branch: "${params.Branch}"
  
 }
}
