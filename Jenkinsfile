pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipStagesAfterUnstable()
  }
  agent {
    docker {
      image 'jojomi/hugo'
    }
  }

  environment {
    YUM_SRV = 'yum.cumulocity.com'
    YUM_USR = 'hudson'
    YUM_RELEASE_PATH = '/var/www/staticpage-new/guides'
  }

  stages {
    stage('Build') {
      steps {
        sh 'hugo'
        sshagent(['hudson-ssh-resources']) {
          sh "rsync -avh ./public/* ${env.YUM_USR}@${env.YUM_SRV}:${env.YUM_DEST_DIR} --delete"
        }
      }
    }
  }
}
