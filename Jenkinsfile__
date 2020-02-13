#! groovy
pipeline {
  agent none
  parameters {
    choice(
      name: "Environment",
      choices: ['staging', 'prod'],
      description : "Release Scope for basic-auth. Deployment for dev needs to be done manually ",
    )
    choice(
      name: "Version",
      choices: ['Major', 'Minor', 'Patch'],
      description : "SemVer-String",
    )
  }
  environment {
    stack_prefix = "${params.Environment == 'prod' ? 'xdn' : 'xdn-staging'}"
    stack_environment = "${params.Environment == 'prod' ? 'prod' : 'stage'}"
    builds_to_keep = "${params.Environment == 'prod' ? '5' : '1'}"
  }
  options {
        buildDiscarder(logRotator(numToKeepStr: env.builds_to_keep))
        disableConcurrentBuilds()
  }
  stages {
    stage('Build') {
      agent {
        docker {
          image 'python:2-alpine'
        }
      }
      steps {
        sh 'python -m py_compile sources/add2vals.py sources/calc.py'
      }
    }
    stage('Test') {
      agent {
        docker {
          image 'qnib/pytest'
        }
      }
      steps {
        sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
      }
      post {
        always {
          junit 'test-reports/results.xml'
        }
      }
    }
    stage('Deliver') {
      agent {
        docker {
          image 'cdrx/pyinstaller-linux:python2'
        }
      }
      steps {
        sh 'pyinstaller --onefile sources/add2vals.py'
      }
      post {
        success {
          archiveArtifacts 'dist/add2vals'
        }
      }
    }
  }
}
