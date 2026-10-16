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
    stage('tool container') {
      steps {
        container('helm') {
          sh 'helm version'
        }
      }
    }
  }
}
