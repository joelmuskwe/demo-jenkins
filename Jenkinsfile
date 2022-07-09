pipeline {
  agent any
  tools {nodejs "Nodev16.3.2"}
  stages {
    stage('Install') {
      steps {
        sh 'npm install -g yarn'
        sh "yarn"
        }
    }

    stage('NX Lint') {
      steps {
        sh 'nx lint --project=demo-app'
        }
    }

    stage('Build') {
      steps {
        sh "nx build --project=demo-app --configuration=production"
        }
    }

    stage('Build') {
      steps { sh 'npm run-script build' }
    }
  }
}
