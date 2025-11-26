node('node-01') {
  stage('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage('Compile-Package') {
    def mvnTool = tool name: 'maven-3.8.6', type: 'Maven'
    
    withEnv(["MAVEN_HOME=${mvnTool}", "PATH+MAVEN=${mvnTool}/bin"]) {
      sh 'mvn package'
    }
  }
}
