node ('node-01'){
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    sh 'echo $PATH' 
    sh 'mvn -version' 
    sh 'mvn package'
   }
}
