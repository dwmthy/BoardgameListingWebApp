node('jdk11') {
  try {
    def mvnHome = tool 'maven-3.8.6'
    def jdk17   = tool name: 'jdk-17', type: 'hudson.model.JDK'

    if(env.BRANCH_NAME.startsWith('dev')) {
        
        stage('SCM Checkout') {
            deleteDir()
            checkout scm
        }
        
        stage('Compile') {
            withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
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

        stage('Noti success'){
            mail to: "duydeptrai2004tv@gmail.com",
            subject: "${JOB_NAME} - Build # ${BUILD_NUMBER} - SUCCESS!",
            body: "Check console output at ${BUILD_URL} to view the results. "
        }
        
    } else if (env.BRANCH_NAME == 'release' || env.BRANCH_NAME.startsWith('uat')) {
        stage('SCM Checkout') {
            deleteDir()
            checkout scm
        }
        
        stage('Compile') {
            withEnv(["JAVA_HOME=${jdk17}", "PATH+JAVA=${jdk17}/bin"]) {
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
     
        stage('Deploy artifact to remote vms') {
            withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                sshagent(['jenkins-user']){
                sh "ssh -o StrictHostKeyChecking=no jenkins@172.31.6.101 'test -f /home/jenkins/deploy/boardgame.jar  && cp /home/jenkins/deploy/boardgame.jar /home/jenkins/deploy/boardgame.jar2.jar || true'"
                sh "ssh jenkins@172.31.6.101 'curl -v -u ${NEXUS_USER}:${NEXUS_PASS} -o /home/jenkins/deploy/boardgame.jar \
                https://nexus.duydinh.online/repository/maven-releases/com.duydinh.app/boardgame/2.0.1/boardgame-2.0.1-${env.BRANCH_NAME}.jar'"
                sh "ssh jenkins@172.31.6.101 'java -jar /home/jenkins/deploy/boardgame.jar &'"   
                }
            }
        }
        
    } else {
      echo "No pipeline configured for branch: ${env.BRANCH_NAME}"
    }
  }
  catch (err) {
    stage('Noti fail'){
        mail to: "duydeptrai2004tv@gmail.com",
        subject: "${JOB_NAME} - Build #${BUILD_NUMBER} - FAILURE!",
        body: "Failure log: ${err.stageLog}. Please check the console output at ${BUILD_URL} for details."
    }
  }
}

  


