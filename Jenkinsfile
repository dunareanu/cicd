pipeline {
  agent { label 'slave' }

  environment {
    GIT_COMMIT  = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
    GIT_TAG     = sh(script: "git tag --contains", returnStdout: true).trim()
  }
  options {
    buildDiscarder logRotator(
      daysToKeepStr: '16',
      numToKeepStr: '10'
    )
  }
  stages  {
    stage("Set environment") {
      steps {
        script {

          echo "${env.GIT_BRANCH}"
          echo "${env.TAG_NAME}"

          WORKDIRECTORY="${env.WORKSPACE}"
          BRANCH_NAME = "${env.GIT_BRANCH.split('/').size() > 1 ? env.GIT_BRANCH.split('/')[1..-1].join('/') : env.GIT_BRANCH}"

          echo "Setting environment for branch ${BRANCH_NAME}"
          if (BRANCH_NAME == 'test-master') {
            DEPLOYMENT_ENV = 'dev'
            IMAGE_VERSION = GIT_COMMIT
          } 
          if (BRANCH_NAME =~ /[0-9].[0-9].[0-9].[0-9]-rc[0-9]+/) {
            DEPLOYMENT_ENV = 'prod'
            IMAGE_VERSION = BRANCH_NAME
          }
          echo "Environment set as ${DEPLOYMENT_ENV}. Image version: ${IMAGE_VERSION}"
        }
      }
    }

    stage("Build Docker Image and push to nexus") {
      steps {
        script {
           
            sh """
              echo $WORKDIRECTORY
              docker run --rm -e TZ=Europe/Bucharest -v m2:/root/.m2 -v ${WORKDIRECTORY}:/workdir -w="/workdir" nexus-private.uae/maven:3.6-jdk-11-slim mvn clean install
              docker build  -t nexus-private.uae/repository/${DEPLOYMENT_ENV}-testing-jenkinsfile:"${IMAGE_VERSION}" .
              docker run --rm -v ${PWD}:/workdir -w="/workdir" -u $(id -u ${USER}):$(id -g ${USER}) nexus-private.uae/maven:3.6-jdk-11-slim mvn --settings /usr/share/maven/.m2/settings.xml sonar:sonar -Dsonar.dependecyCheck.htmlReportPath=./target/dependency-check-report.html -Dsonar.dependecyCheck.jsonReportPath=./target/dependency-check-report.json -Dsonar.host.url=https://gitlab.private.uae/sonar -Dsonar.login=user -Dsonar.password=encryptedpassword
              docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${PWD}:/workdir nexus-private.uae/aquasec/trivy:0.20.2 fs --security-checks config /workdir/
              docker push nexus-private.uae/repository/${DEPLOYMENT_ENV}-testing-jenkinsfile:${IMAGE_VERSION}
            """
          
        }
      }
    }

     stage('Deploy DEV') {
      steps {
        sh '''#!/usr/bin/env bash
          echo "Shell Process ID: $$"
          git config --global user.email "ci@ci.com"
          git config --global user.name "jenkins-ci"
          rm -rf argocd-k8s-manifest
          git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/argocd-k8s-manifest.git
          cd ./argocd-k8s-manifest/overlays/dev && sed -i "s/image: nexus-private.uae\/repository\/images-dev\/example-microservice:.*/image: nexus-private.uae\/repository\/images-dev\/example-microservice:${TAG_NAME}/" example-microservice/deployment.yaml
          git commit -am 'Publish new version' && git push 
        '''
      }
    }
     stage('Deploy PROD') {
      steps {
        input message:'Approve deployment?'
        sh '''#!/usr/bin/env bash
          cd ./argocd-k8s-manifest/overlays/prod && sed -i "s/image: nexus-private.uae\/repository\/images-prod\/example-microservice:.*/image: nexus-private.uae\/repository\/images-prod\/example-microservice:${TAG_NAME}/" example-microservice/deployment.yaml
          git commit -am 'Publish new version' && git push 
        '''
      }
    }
  }
  }
  post {
    always {
      echo "One way or another, I have finished"
      //deleteDir() /* clean up our workspace */
    }
    // trigger when successful
    success {
      echo "I succeeded!"
      slackSend (channel: "${env.SLACK_CHANNEL}", color: "${env.SLACK_COLOR_GOOD}", message: "*SUCCESS:* Job '${env.JOB_NAME} (${env.BUILD_URL})")
    }
    // trigger when failed
    failure {
        echo "I failed :("
        slackSend (channel: "${env.SLACK_CHANNEL}", color: "${env.SLACK_COLOR_DANGER}", message: "*FAILED:* Job '${env.JOB_NAME} (${env.BUILD_URL})")
    }

    // trigger when aborted
    aborted {
        echo "Build aborted!"
        slackSend (channel: "${env.SLACK_CHANNEL}", color: "${env.SLACK_COLOR_WARNING}", message: "*ABORTED:* Job '${env.JOB_NAME} (${env.BUILD_URL})")
    }
  }
} 