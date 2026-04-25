node {
    stage('SCM Checkout'){
        deleteDir()
        checkout scm
    }

    stage('Compile') {
        def mvnHome = tool 'maven-3.8.6'
        sh "${mvnHome}/bin/mvn compile"
    }


    stage('Test') {
        def mvnHome = tool 'maven-3.8.6'
        sh "${mvnHome}/bin/mvn test"
    }

    stage('Sonarqube Scan'){
        withSonarQubeEnv('sonarqube-server') {
            sh 'mvn sonar:sonar'
        }
    }
    
    tage('SonarQube Scan') {
            withSonarQubeEnv('sonarqube-server') {
                sh 'mvn sonar:sonar'
            }
        }
    
    stage('Quality Gate') {
        timeout(time: 2, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline failed due to Quality Gate: ${qg.status}"
            }
        }
    }
}
