pipeline {

  agent {
    // Node setup : minimal centos7, plugged into Jenkins, and
    // git config --global http.sslVerify false
    // sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm
    // sudo yum -y install python36u python36u-pip python36u-devel git curl gcc
    // git config --global http.sslVerify false
    // sudo curl -fsSL get.docker.com | bash
    label 'hf-t-build02-docker'
  }

  stages {

    stage ('Get latest code') {
      steps {
        checkout scm
      }
    }

    stage ('Setup Python virtual environment') {
      steps {
        sh ''' #!/bin/bash
          python3 -m venv virtenv
          . ./virtenv/bin/activate
          python3 -m pip install --upgrade ansible molecule docker
          pip3 install 'molecule[docker]'
        '''
      }
    }

    stage ('Display versions') {
      steps {
        sh '''
          . ./virtenv/bin/activate
          docker -v
          python -V
          ansible --version
          molecule --version
        '''
      }
    }

    stage ('Molecule test') {
      steps {
        sh '''
          . ./virtenv/bin/activate
          molecule test
        '''
      }
    }

  }

}
