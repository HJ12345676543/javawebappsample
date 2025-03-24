import groovy.json.JsonSlurper

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=8f05bf3a-7fc0-4937-a58d-93380154e18d',
    'AZURE_TENANT_ID=aa05f5be-0265-4c8a-ae7f-5f78a08d388c'
  ]) {

    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'INFO540-RG-Compute'
      def webAppName = 'jenkins-demo-haijun'

      withCredentials([
        usernamePassword(
          credentialsId: 'AzureServicePrincipal',
          usernameVariable: 'AZURE_CLIENT_ID',
          passwordVariable: 'AZURE_CLIENT_SECRET'
        )
      ]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      sh "az webapp deploy --resource-group $resourceGroup --name $webAppName --src-path target/calculator-1.0.war --type war"

      sh 'az logout'
    }
  }
}
