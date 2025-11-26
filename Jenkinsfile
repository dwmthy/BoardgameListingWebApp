node('node-01'){
    stage('SCM Checkout'){
        git 'https://github.com/dwmthy/BoardgameListingWebApp'
    }

    stage('Compile - package'){
        def mvnHome = tool name: 'maven-3.8.6', type: 'maven'
        withEnv(["PATH+MAVEN=${mvnHome}/bin"]) {
            sh 'mvn package'
        }
    }
}
