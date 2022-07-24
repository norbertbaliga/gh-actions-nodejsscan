pipeline {
  agent {
    node {
      label 'centos'
    }
  }
  stages {
    stage('Build docker image') {
      steps {
        sh 'echo "Build image for the app: ${JOB_NAME}"'
        sh 'docker build -t ${JOB_NAME} -f ./Dockerfile .'
      }
    }

    stage('Generate SBOM with Syft') {
      steps {
        sh 'echo "Generate SBOM with Syft: ${JOB_NAME}-sbom.xml"'
        sh 'syft docker:${JOB_NAME} -o cyclonedx-xml --file ${JOB_NAME}-sbom.xml'
      }
    }

    stage('Process SBOM') {
      parallel {
        stage('Upload SBOM to Dependency-track') {
          steps {
            sh 'echo "Upload SBOM XML to Dependency-track"'
            withCredentials(bindings: [string(credentialsId: '9443aca9-48ed-48ac-bf1c-c572a6a9f74d', variable: 'API_KEY')]) {
              dependencyTrackPublisher(dependencyTrackUrl: 'http://192.168.99.1:8081', dependencyTrackFrontendUrl: 'http://192.168.99.1:8080', artifact: '${JOB_NAME}-sbom.xml', projectName: 'Jenkins-vuln-NodeJS-app', projectVersion: '1.0', autoCreateProjects: true, synchronous: true, dependencyTrackApiKey: API_KEY)
            }
          }
        }

        stage('Run Grype scan') {
          steps {
            sh 'echo "Run Grype scans with SBOM"'
            sh 'grype sbom:${JOB_NAME}-sbom.xml -o table --file ${JOB_NAME}-grype_table.txt'
            sh 'grype sbom:${JOB_NAME}-sbom.xml -o sarif --file ${JOB_NAME}-grype_sarif.json'
          }
        }
      }
    }

  }
  post {
    always {
      step $class: 'ArtifactArchiver', artifacts: "${JOB_NAME}-*", allowEmptyArchive: true, followSymlinks: false
    }
  }
  options {
    buildDiscarder(logRotator(daysToKeepStr: '', numToKeepStr: '10'))
  }
}