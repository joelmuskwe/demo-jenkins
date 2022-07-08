class Configuration {
    String APP_NAME
    String DEPLOYMENT_ENVIRONMENT
    String PROXY_PASS
    String COMMAND

    Configuration(APP_NAME, DEPLOYMENT_ENVIRONMENT, PROXY_PASS, COMMAND) {
        this.APP_NAME = APP_NAME
        this.DEPLOYMENT_ENVIRONMENT = DEPLOYMENT_ENVIRONMENT
        this.PROXY_PASS = PROXY_PASS
        this.COMMAND = COMMAND
    }
}

def getConfiguration(String branch) {
    if(branch.matches("develop")) {
        return new Configuration("rcms", "develop", "192.168.150.60:8080", "development")
    } else if (branch.matches("deployments/staging")) {
        return new Configuration("rcms", "staging", "192.168.150.60:8080", "staging")
  } else if (branch.matches("main")) {
        return new Configuration("rcms", "live", "192.168.150.60:8080", "production")
  } else if(branch.matches("^deployments/.*/(develop|staging|live)")) {
        def split = branch.split("/")
        return new Configuration(split[1], split[2], "192.168.150.60:8080", split[1] + split[2].capitalize())
    }
}

pipeline {
  agent: any
  tools {
    nodejs "Nodev16.3.2"
  }
    environment {
      CONFIGURATION = getConfiguration(BRANCH_NAME)
    }
  stages {
    stage("Node Install Packages") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        sh "npm install -g yarn"
        sh "yarn"
      }
    }
    stage("Node Build") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        sh "nx build --project=${env.CONFIGURATION.APP_NAME} --configuration=${env.CONFIGURATION.COMMAND}"
      }
    }
    stage("Node Test") {
      condition {
        equals("BRANCH", "develop|deployments/*/develop")
      }
      steps {
        sh "nx test --project=${env.CONFIGURATION.APP_NAME}"
      }
    }
    stage("Node Lint") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        sh "nx lint --project=${env.CONFIGURATION.APP_NAME}"
      }
    }
    stage("Docker Build") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        // http://192.168.150.60:8080
        sh "sed -i 's/PROXY_PASS/${env.CONFIGURATION.PROXY_PASS}/g' ./nginx.conf"
        sh "docker build -f ./Dockerfile -t docker.registry/${env.CONFIGURATION.APP_NAME}-${env.CONFIGURATION.CLIENT_NAME}-${env.CONFIGURATION.DEPLOYMENT_ENVIRONMENT}-portal-hybrid:${BUILD_NUMBER} ."
      }
    }
    stage("Docker Push") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        sh "docker push docker.registry/${env.CONFIGURATION.APP_NAME}-${env.CONFIGURATION.CLIENT_NAME}-${env.CONFIGURATION.DEPLOYMENT_ENVIRONMENT}-portal-hybrid:${BUILD_NUMBER}"
      }
    }
    stage("Deploy to Kubernetes") {
      condition {
        equals("BRANCH", "main|develop|deployments/*")
      }
      steps {
        sh "sed -i 's/APP_NAME/${env.CONFIGURATION.APP_NAME}/g' ./kubernetes-deployment.yaml"
        sh "sed -i 's/CLIENT_NAME/${env.CONFIGURATION.CLIENT_NAME}/g' ./kubernetes-deployment.yaml"
        sh "sed -i 's/DEPLOYMENT_ENVIRONMENT/${env.CONFIGURATION.DEPLOYMENT_ENVIRONMENT}/g' ./kubernetes-deployment.yaml"
        sh "sed -i 's/BUILD_NUMBER/${env.CONFIGURATION.BUILD_NUMBER}/g' ./kubernetes-deployment.yaml"
        sh "kubectl apply -f ./kubernetes-deployment.yaml"
      }
    }
  }
}
