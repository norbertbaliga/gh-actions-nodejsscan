pipeline {
  agent {
    node { 
        label 'centos'
    } 
  }
  stages {
    stage('Build docker image') {
      steps {
        sh 'docker build -t vuln-nodejs-app:latest -f ./Dockerfile .'
      }
    }

    stage('Generate SBOM with Syft') {
      steps {
        sh 'syft docker:vuln-nodejs-app:latest -o cyclonedx-xml --file vuln-nodejs-app_syft-sbom.xml'
      }
    }

    stage('Upload SBOM to Dependency-tracker') {
      steps {
        withCredentials([string(credentialsId: '9443aca9-48ed-48ac-bf1c-c572a6a9f74d', variable: 'API_KEY')]) {
          dependencyTrackPublisher dependencyTrackUrl: 'http://192.168.99.1:8081', dependencyTrackFrontendUrl: 'http://192.168.99.1:8080',artifact: 'vuln-nodejs-app_syft-sbom.xml', projectName: 'Jenkins-vuln-NodeJS-app2', projectVersion: '1.0', autoCreateProjects: true, synchronous: true, dependencyTrackApiKey: API_KEY
        }
      }
    }
  }
  post {
    always {
      step($class: 'ArtifactArchiver', artifacts: 'vuln-nodejs-app_syft-sbom.xml', allowEmptyArchive: true, followSymlinks: false)
    }

  }
  options {
    buildDiscarder(logRotator(daysToKeepStr: '', numToKeepStr: '10'))
  }
}