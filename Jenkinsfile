node('jdk11') {
  try {
    def mvnHome = tool 'maven-3.8.6'
    def jdk17   = tool name: 'jdk-17', type: 'hudson.model.JDK'
    def baseVersion   // read from pom.xml
    def GIT_HASH 
    def safeBranch

    stage('SCM Checkout') {
        deleteDir()
        checkout scm
        // Read base version from pom.xml
        baseVersion = sh(
        script: "${mvnHome}/bin/mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
        returnStdout: true
        ).trim().replace('-SNAPSHOT', '')
        // baseVersion = "1.2.0"

        GIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        safeBranch = env.BRANCH_NAME.split('/')[0].replaceAll('[^a-zA-Z0-9._-]', '-')    }

    if (env.CHANGE_ID) {
      // This is a PR build — env.CHANGE_TARGET tells you where it's merging into
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

      stage('Noti PR success') {
        mail to: "duydeptrai2004tv@gmail.com",
          subject: "${JOB_NAME} - PR #${env.CHANGE_ID} → ${env.CHANGE_TARGET} - SUCCESS!",
          body: "PR passed all checks. Review at ${BUILD_URL}"
      }

    } else if (env.BRANCH_NAME.startsWith('dev')) {
        
        if (env.CHANGE_ID) {
            echo "Skipping dev branch build — already running as PR-${env.CHANGE_ID}"
            return
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
            echo "Mail to success"
            // mail to: "duydeptrai2004tv@gmail.com",
            // subject: "${JOB_NAME} - Build # ${BUILD_NUMBER} - SUCCESS!",
            // body: "Check console output at ${BUILD_URL} to view the results. "

        }
        
    } else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME.startsWith('uat')) {

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

        if(env.BRANCH_NAME == 'main'){
            stage('Push artifact to Nexus Repo') {
                withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                sh """curl -v -u ${NEXUS_USER}:${NEXUS_PASS} --upload-file target/*.jar \
                    https://nexus.duydinh.online/repository/maven-releases/com/duydinh/app/boardgame/${baseVersion}/boardgame-${baseVersion}-${safeBranch}.jar
                """
                }
            }

            stage('Deploy artifact to remote vms') {
                withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sshagent(['jenkins-user']){
                    sh "ssh -o StrictHostKeyChecking=no jenkins@172.31.5.52 'test -f /home/jenkins/deploy/boardgame.jar  && cp /home/jenkins/deploy/boardgame.jar /home/jenkins/deploy/boardgame.jar2.jar || true'"
                    sh "ssh jenkins@172.31.5.52 'curl -v -u ${NEXUS_USER}:${NEXUS_PASS} -o /home/jenkins/deploy/boardgame.jar \
                    https://nexus.duydinh.online/repository/maven-releases/com/duydinh/app/boardgame/${baseVersion}/boardgame-${baseVersion}-${safeBranch}.jar'"
                    sh "ssh jenkins@172.31.5.52 'nohup java -jar /home/jenkins/deploy/boardgame.jar > /home/jenkins/deploy/app.log 2>&1 &'"                     
                    }
                }
            }
        } else {
            stage('Push artifact to Nexus Repo') {
                withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                sh """curl -v -u ${NEXUS_USER}:${NEXUS_PASS} --upload-file target/*.jar \
                    https://nexus.duydinh.online/repository/maven-releases/com/duydinh/app/boardgame/${baseVersion}/boardgame-${baseVersion}-${safeBranch}-${GIT_HASH}.jar
                """
                }
            }
             
            stage('Deploy artifact to remote vms') {
                withCredentials([usernamePassword(credentialsId: "nexus-cred", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sshagent(['jenkins-user']){
                    sh "ssh -o StrictHostKeyChecking=no jenkins@172.31.6.101 'test -f /home/jenkins/deploy/boardgame.jar  && cp /home/jenkins/deploy/boardgame.jar /home/jenkins/deploy/boardgame.jar2.jar || true'"
                    sh "ssh jenkins@172.31.6.101 'curl -v -u ${NEXUS_USER}:${NEXUS_PASS} -o /home/jenkins/deploy/boardgame.jar \
                    https://nexus.duydinh.online/repository/maven-releases/com/duydinh/app/boardgame/${baseVersion}/boardgame-${baseVersion}-${safeBranch}-${GIT_HASH}.jar'"
                    sh "ssh jenkins@172.31.6.101 'nohup java -jar /home/jenkins/deploy/boardgame.jar > /home/jenkins/deploy/app.log 2>&1 &'" 
                    }
                }
            }          
        }
        
        stage('Health Check') {
            sleep 30
            retry(3) {
            def r = httpRequest(
                url: "http://localhost:8080",
                validResponseCodes: '200:399',
                timeout: 10
            )
            echo "Health status: ${r.status}"
            sleep 5
            }
        }

        stage('Noti success') {
            echo "Mail to success"
        }

    } else {
      echo "No pipeline configured for branch: ${safeBranch}"
    }
  }
  catch (err) {
    stage('Noti fail'){
        echo "Mail to fail"
        // mail to: "duydeptrai2004tv@gmail.com",
        // subject: "${JOB_NAME} - Build #${BUILD_NUMBER} - FAILURE!",
        // body: "Error: ${err.getMessage()}. Please check the console output at ${BUILD_URL} for details."
    }
    throw err
  }
}

  


