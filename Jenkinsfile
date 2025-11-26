node ('node-01'){
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    sh 'mvh package'
  }
}
