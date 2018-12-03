#!/usr/bin/env groovy

pipeline {
  agent none
  environment {
    BASE_DIR="src/github.com/elastic/apm-pipeline-library"
    JOB_GIT_CREDENTIALS = "f6c7695a-671e-4f4f-a331-acdce44ff9ba"
  }
  options {
    timeout(time: 1, unit: 'HOURS') 
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20', daysToKeepStr: '30'))
    timestamps()
    ansiColor('xterm')
    disableResume()
    durabilityHint('PERFORMANCE_OPTIMIZED')
  }
  stages {
    stage('Initializing'){
      agent { label 'linux && immutable' }
      options { skipDefaultCheckout() }
      environment {
        PATH = "${env.PATH}:${env.WORKSPACE}/bin"
      }
      stages {
        /**
         Checkout the code and stash it, to use it on other stages.
        */
        stage('Checkout') {
          steps {
            gitCheckout(basedir: "${BASE_DIR}")
            dir("${BASE_DIR}"){
              sh """#!/bin/bash
              MVNW_VER="maven-wrapper-0.4.2"
              MVNW_DIR="maven-wrapper-\${MVNW_VER}"
              curl -sLO "https://github.com/takari/maven-wrapper/archive/\${MVNW_VER}.tar.gz"
              tar -xzf "\${MVNW_VER}.tar.gz"
              mv "\${MVNW_DIR}/.mvn/" .
              mv "\${MVNW_DIR}/mvnw" .
              mv "\${MVNW_DIR}/mvnw.cmd" .
              rm -fr "\${MVNW_DIR}"
              """
            }
            stash allowEmpty: true, name: 'source', useDefaultExcludes: false
          }
        }
        /**
         Checkout the code and stash it, to use it on other stages.
        */
        stage('Test') {
          steps {
            withEnvWrapper() {
              unstash 'source'
              dir("${BASE_DIR}"){
                sh './mvnw clean test --batch-mode -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn'
              }
            }
          }
          post { 
            always { 
              junit(allowEmptyResults: true, 
                keepLongStdio: true, 
                testResults: "${BASE_DIR}/target/surefire-reports/junit-*.xml,${BASE_DIR}/target/surefire-reports/TEST-*.xml")
            }
          }
        }
      }
    }
  }
  post { 
    success {
      echoColor(text: '[SUCCESS]', colorfg: 'green', colorbg: 'default')
    }
    aborted {
      echoColor(text: '[ABORTED]', colorfg: 'magenta', colorbg: 'default')
    }
    failure { 
      echoColor(text: '[FAILURE]', colorfg: 'red', colorbg: 'default')
      //step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: "${NOTIFY_TO}", sendToIndividuals: false])
    }
    unstable { 
      echoColor(text: '[UNSTABLE]', colorfg: 'yellow', colorbg: 'default')
    }
  }
}