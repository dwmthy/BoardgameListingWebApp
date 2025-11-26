node{
  stage ('SCM Checkout') {
    git 'https://github.com/dwmthy/BoardgameListingWebApp.git'
  }

  stage ('Compile-Package'){
    sh 'mvh package'
  }
}
