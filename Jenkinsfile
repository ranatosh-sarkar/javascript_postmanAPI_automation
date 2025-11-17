pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '15'))
    timeout(time: 20, unit: 'MINUTES')
  }

  environment {
    NEWMAN_ALLURE_RESULTS = 'allure-results'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Node setup & deps') {
      steps {
        bat 'node -v & npm -v'
        bat 'npm ci'
      }
    }

    stage('Clean old reports') {
      steps {
        bat '''
          if exist newman (
            echo Deleting old Newman reports...
            rmdir /s /q newman
          )
          if exist allure-report (
            echo Deleting old generated Allure HTML...
            rmdir /s /q allure-report
          )
          if exist allure-results (
            echo Deleting old Allure raw results...
            rmdir /s /q allure-results
          )
        '''
      }
    }

    stage('Run Newman (QA)') {
      steps {
        script {
          int code = bat(
            returnStatus: true,
            label: 'Newman QA run',
            script: 'npm run test:qa'
          )
          if (code != 0) {
            echo "Newman exited with code ${code} – marking build UNSTABLE so reports are still generated."
            currentBuild.result = 'UNSTABLE'
          }
        }
      }
    }

    stage('Debug: list Allure result files') {
      steps {
        bat """
          echo ===== Listing %NEWMAN_ALLURE_RESULTS% =====
          if exist %NEWMAN_ALLURE_RESULTS% (
            dir %NEWMAN_ALLURE_RESULTS%
          ) else (
            echo Folder %NEWMAN_ALLURE_RESULTS% does not exist
          )
        """
      }
    }
  }

  post {
    always {
      junit allowEmptyResults: true,
            testResults: 'newman/reports/junit/*.xml'

      allure includeProperties: false,
             jdk: '',
             results: [[path: "${env.NEWMAN_ALLURE_RESULTS}"]]

      archiveArtifacts artifacts: 'newman/reports/**',
                       fingerprint: true,
                       allowEmptyArchive: true

      publishHTML(target: [
        reportDir: 'newman/reports/html',
        reportFiles: 'qa-run.html',
        reportName: 'Newman HTML Report',
        keepAll: true,
        alwaysLinkToLastBuild: true,
        allowMissing: true
      ])
    }
  }
}
