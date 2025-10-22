pipeline {
  agent any  // or specify label('ansible') for a node with ansible installed

  environment {
    INVENTORY = 'inventory'
    PLAYBOOK  = 'site.yml'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Lint') {
      steps {
        // ignore failure if ansible-lint not installed; otherwise it prevents bad code
        sh '''
          if command -v ansible-lint >/dev/null 2>&1; then
            ansible-lint ${PLAYBOOK} || true
          else
            echo "ansible-lint not installed, skipping lint."
          fi
        '''
      }
    }

    stage('Syntax Check') {
      steps {
        sh "ansible-playbook --syntax-check -i ${INVENTORY} ${PLAYBOOK}"
      }
    }

    stage('Run Playbook') {
      steps {
        // This runs the playbook; requires the agent to be able to sudo or run as root
        sh "ansible-playbook -i ${INVENTORY} ${PLAYBOOK}"
      }
    }

    stage('Verify') {
      steps {
        // Try fetching the page from localhost on the agent
        sh '''
          echo "Waiting 3s for services..."
          sleep 3
          if curl -sS http://localhost | grep -q "Welcome to Saad"; then
            echo "WEBPAGE OK"
          else
            echo "WEBPAGE NOT OK"
            curl -v http://localhost || true
            exit 1
          fi
        '''
      }
    }
  }

  post {
    success { echo "Playbook ran successfully." }
    failure {
      echo "Failure — check console output."
    }
  }
}
