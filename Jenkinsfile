node {
  try {
    stage('Checkout') {
      checkout scm
    }
    stage('Environment') {
      sh 'git --version'
      echo "Branch: ${env.BRANCH_NAME}"
      sh 'docker -v'
      sh 'printenv'
    }
    stage('Build Docker'){
     if (env.BRANCH_NAME == 'feature') {
     sh 'docker build -t hamsa20/docker-react -f Dockerfile.dev .'
     }
     else{
        sh 'docker build -t hamsa20/docker-react -f Dockerfile .'
      }
    }
    stage('Docker test'){
      if (env.BRANCH_NAME == 'feature') {
      sh 'docker run -e CI=true hamsa20/docker-react npm run test'
      }
      else{
        echo "We are in ${env.BRANCH_NAME} branch. Skip Docker test stage"
      }
    }
    stage('Push Image'){
      withCredentials([azureServicePrincipal('02721850-4b8f-4fd8-a10d-ba3962133797')]) {
      sh '''
        az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}        
        az acr login --name 'Jenkinshamsa'
        docker tag hamsa20/docker-react:latest jenkinshamsa.azurecr.io/docker-react:latest
        docker push jenkinshamsa.azurecr.io/docker-react:latest
      '''
      }
      //app = docker.build('jenkinshamsa.azurecr.io/docker-react')
      //docker.withRegistry('https://jenkinshamsa.azurecr.io', 'ACR'){
       //   app.push("${env.BUILD_NUMBER}")
        //  app.push('latest')
       //}
    }
    stage('Deploy into Web App'){
      withCredentials([azureServicePrincipal('02721850-4b8f-4fd8-a10d-ba3962133797')]) {
       sh 'az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}'
      }
      withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
       sh 'az webapp config container set --name DockerFrontendApp --resource-group Azure-HK --docker-custom-image-name jenkinshamsa.azurecr.io/docker-react:latest --docker-registry-server-url https://jenkinshamsa.azurecr.io --docker-registry-server-user ${username} --docker-registry-server-password ${password}'
      }
    }    
  }
  catch (err) {
    throw err
  }
}
