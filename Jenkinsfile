node ('node-01'){
   environment {
     MAVEN_HOME = '/opt/apache-maven-3.8.6'
     PATH = "${MAVEN_HOME}/bin:${env.PATH}"
  }
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    sh 'mvn package'
   }
}
