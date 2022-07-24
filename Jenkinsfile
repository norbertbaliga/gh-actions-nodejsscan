pipeline {
  agent {
    node {
      label 'centos'
    }

  }
  stages {
    stage('Build docker image') {
      steps {
        sh 'echo \'Build image for the app\''
        sh 'docker build -t $IMAGE_NAME -f ./Dockerfile .'
      }
    }

    stage('Generate SBOM with Syft') {
      steps {
        sh 'echo \'Generate SBOM with Syft\''
        sh 'syft docker:$IMAGE_NAME -o cyclonedx-xml --file $SBOM_FILE'
      }
    }

    stage('Use SBOM') {
      parallel {
        stage('Upload SBOM to Dependency-track') {
          steps {
            sh 'echo \'Upload SBOM XML to Dependency-track\''
            withCredentials(bindings: [string(credentialsId: '9443aca9-48ed-48ac-bf1c-c572a6a9f74d', variable: 'API_KEY')]) {
              dependencyTrackPublisher(dependencyTrackUrl: 'http://192.168.99.1:8081', dependencyTrackFrontendUrl: 'http://192.168.99.1:8080', artifact: '$SBOM_FILE', projectName: 'Jenkins-vuln-NodeJS-app', projectVersion: '1.0', autoCreateProjects: true, synchronous: true, dependencyTrackApiKey: API_KEY)
            }
          }
        }

        stage('Run Grype scan') {
          steps {
            sh 'echo \'Run Grype scans with SBOM\''
            sh 'grype sbom:$SBOM_FILE -o table --file $GRYPE_FILE-table.txt'
            sh 'grype sbom:$SBOM_FILE -o sarif --file $GRYPE_FILE-sarif.json'
          }
        }
      }
    }

  }
  environment {
    IMAGE_NAME = 'vuln-nodejs-app:latest'
    SBOM_FILE = 'vuln-nodejs-app_syft-sbom.xml'
    GRYPE_FILE = 'vuln-nodejs-app_grype'
  }
  post {
    always {
      step $class: 'ArtifactArchiver', artifacts: '$SBOM_FILE, $GRYPE_FILE*', allowEmptyArchive: true, followSymlinks: false
    }

  }
  options {
    buildDiscarder(logRotator(daysToKeepStr: '', numToKeepStr: '10'))
  }
}