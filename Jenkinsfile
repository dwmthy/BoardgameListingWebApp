node ('node-01'){
  tools { 
    maven 'maven-3.8.6' 
  }
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    sh 'mvn package'
   }
}
