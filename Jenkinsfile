import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv([
    'AZURE_SUBSCRIPTION_ID=8f05bfa3-7fc0-4937-a58d-93380154e18d',
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

      withCredentials([usernamePassword(
        credentialsId: 'AzureServicePrincipal',
        passwordVariable: 'AZURE_CLIENT_SECRET',
        usernameVariable: 'AZURE_CLIENT_ID'
      )]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)

      sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
      sh 'az logout'
    }

  } // withEnv
} // node
