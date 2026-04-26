node('jdk11') {
  stage('SCM Checkout') {
    deleteDir()
    checkout scm
  }

  def mvnHome = tool 'maven-3.8.6'
  def jdk17   = tool name: 'jdk-17', type: 'hudson.model.JDK'

  stage('Compile') {
    withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
      sh "${mvnHome}/bin/mvn -version"
      sh "${mvnHome}/bin/mvn -U clean compile"
    }
  }

  stage('Test') {
    withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
      sh "${mvnHome}/bin/mvn test"
    }
  }

  stage('Sonarqube Scan') {
    withSonarQubeEnv('sonarqube-server') {
      withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
        sh "${mvnHome}/bin/mvn sonar:sonar"
      }
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
