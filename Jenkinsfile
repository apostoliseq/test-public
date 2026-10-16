@Library('TestLibrary') _

pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: helm
      image: alpine/helm:3.20.0
      command: ['cat']
      tty: true
'''
    }
  }
  stages {
    stage('default container') {
      steps {
        greet('Peon')
        sh 'echo "I am in: $(hostname)"; which helm || echo "no helm in this container"'
      }
    }
    stage('secret') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'apostolis-cred',
                                          usernameVariable: 'CRED_USER',
                                          passwordVariable: 'CRED_PASS')]) {
          sh 'echo "1 plain:  $CRED_USER / $CRED_PASS"'
          sh 'echo "2 base64: $(echo -n "$CRED_PASS" | base64)"'
          sh 'echo "3 first4: $(echo -n "$CRED_PASS" | cut -c1-4)"'
        }
        sh 'echo "4 outside: [$CRED_PASS]"'
      }
    }

    stage('tool container') {
      steps {
        container('helm') {
          sh 'helm version'
        }
      }
    }
  }
}
