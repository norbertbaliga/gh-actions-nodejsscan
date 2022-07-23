pipeline {
  environment {
    IMAGE_NAME = "vuln-nodejs-app:latest"
    SBOM_FILE = "vuln-nodejs-app_syft-sbom.xml"
  }
  agent {
    node { 
        label 'centos'
    } 
  }
  stages {
    stage('Build docker image') {
      steps {
        sh 'docker build -t $IMAGE_NAME -f ./Dockerfile .'
      }
    }

    stage('Generate SBOM with Syft') {
      steps {
        sh 'syft docker:$IMAGE_NAME -o cyclonedx-xml --file $SBOM_FILE'
      }
    }

    stage('Upload SBOM to Dependency-tracker') {
      steps {
        withCredentials([string(credentialsId: '9443aca9-48ed-48ac-bf1c-c572a6a9f74d', variable: 'API_KEY')]) {
          dependencyTrackPublisher dependencyTrackUrl: 'http://192.168.99.1:8081', dependencyTrackFrontendUrl: 'http://192.168.99.1:8080',artifact: '$SBOM_FILE', projectName: 'Jenkins-vuln-NodeJS-app2', projectVersion: '1.0', autoCreateProjects: true, synchronous: true, dependencyTrackApiKey: API_KEY
        }
      }
    }
  }
  post {
    always {
      step($class: 'ArtifactArchiver', artifacts: '$SBOM_FILE', allowEmptyArchive: true, followSymlinks: false)
    }

  }
  options {
    buildDiscarder(logRotator(daysToKeepStr: '', numToKeepStr: '10'))
  }
}