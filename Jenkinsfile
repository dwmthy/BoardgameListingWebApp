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
    timeout(time: 1, unit: 'HOURS') {
      def qg = waitForQualityGate()
      if (qg.status != 'OK') {
        error "Pipeline failed due to Quality Gate: ${qg.status}"
      }
    }
  }

  stage('Package') {
    withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
      sh "${mvnHome}/bin/mvn package"
    }
  }

  stage('Push artifact to Nexus Repo') {
    withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
      sh """curl -v -u ${NEXUS_USER}:${NEXUS_PASS} --upload-file target/*.jar \
        https://nexus.duydinh.online/repository/maven-releases/com.duydinh.app/boardgame/2.0.1/boardgame-2.0.1-${env.BRANCH_NAME}.jar
      """
    }
  }
}
