pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '15'))
    timeout(time: 20, unit: 'MINUTES')
  }

  environment {
    // Single source of truth for where Newman writes Allure results
    NEWMAN_ALLURE_RESULTS = 'newman/reports/allure-results'
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
        // Clean any previous run artefacts in the WORKSPACE
        bat '''
          if exist newman (
            echo Deleting old Newman reports...
            rmdir /s /q newman
          )
          if exist allure-report (
            echo Deleting old generated Allure HTML...
            rmdir /s /q allure-report
          )
        '''
      }
    }

    stage('Run Newman (QA)') {
      steps {
        script {
          // Run Newman; if it fails, mark build UNSTABLE but still publish reports
          int code = bat(
            returnStatus: true,
            label: 'Newman QA run',
            script: 'npm run test:qa'
          )

          if (code != 0) {
            echo "Newman exited with code ${code} – tests failed, marking build UNSTABLE so reports are still generated."
            currentBuild.result = 'UNSTABLE'
          }
        }
      }
    }

    // Optional debug – shows exactly what Allure will read
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
      // JUnit trend from Newman’s JUnit reporter
      junit allowEmptyResults: true,
            testResults: 'newman/reports/junit/*.xml'

      // Allure plugin: generate HTML from Newman’s raw Allure results
      allure includeProperties: false,
             jdk: '',
             results: [[path: "${env.NEWMAN_ALLURE_RESULTS}"]]

      // Archive all Newman artefacts (HTML, JSON, JUnit, Allure raw)
      archiveArtifacts artifacts: 'newman/reports/**',
                       fingerprint: true,
                       allowEmptyArchive: true

      // Publish the htmlextra HTML report as a Jenkins HTML report
      publishHTML(target: [
        reportDir: 'newman/reports/html',
        reportFiles: 'qa-run.html',
        reportName: 'Newman HTML Report',
        keepAll: true,
        alwaysLinkToLastBuild: true,
        allowMissing: true
      ])
    }

    success {
      echo '✅ Newman tests completed successfully.'
    }

    unstable {
      echo '⚠️ Newman reported test failures – check Allure / HTML / JUnit reports for details.'
    }

    failure {
      echo '❌ Pipeline failed before or during Newman execution.'
    }
  }
}
