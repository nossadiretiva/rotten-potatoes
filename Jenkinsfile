pipeline {
  agent any
  stages {
    stage('Git') {
      steps {
        git(url: 'git@github.com:atlants-development/betlabs-api.git', branch: 'main', credentialsId: 'GitSSH')
      }
    }

  }
}