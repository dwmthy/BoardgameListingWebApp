node ('node-01'){
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    def mvnTool = tool 'maven-3.8.6'
          withEnv(["MAVEN_HOME=${mvnTool}"]) {
          sh 'mvn package'              
     }
   }
}
