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

  }
  post {
    always {
      step($class: 'ArtifactArchiver', artifacts: 'vuln-nodejs-app_syft-sbom.xml', allowEmptyArchive: true, followSymlinks: false)
      echo 'No converter for Publisher: org.jenkinsci.plugins.DependencyTrack.DependencyTrackPublisher'
    }

  }
  options {
    buildDiscarder(logRotator(daysToKeepStr: '0', numToKeepStr: '10'))
  }
}