import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=0279f7da-18d7-44ca-afac-a748c4fef618',
           'AZURE_TENANT_ID=20c807cb-78eb-4dd6-a603-1d4f1e5c7df0']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'weishenfang'
      
      // Login to Azure (with quoted password to handle spaces)
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u "$AZURE_CLIENT_ID" -p "'$AZURE_CLIENT_SECRET'" -t "$AZURE_TENANT_ID"
          az account set -s "$AZURE_SUBSCRIPTION_ID"
        '''
      }
      
      // Get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)
      
      // Upload package via FTP
      // sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      sh "az webapp deploy --resource-group jenkins-get-started-rg --name weishenfang --src-path target/calculator-1.0.war --type war"
      // Log out
      sh 'az logout'
    }
  }
}

