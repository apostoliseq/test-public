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
      volumeMounts:
        - name: lab-kubeconfig
          mountPath: /root/.kube
        - name: lab-signing-key
          mountPath: /tmp/
        - name: lab-signing-pub
          mountPath: /pub
  volumes:
    - name: lab-kubeconfig
      secret:
        secretName: lab-kubeconfig
        defaultMode: 0400
    - name: lab-signing-key
      secret:
        secretName: lab-signing-key
    - name: lab-signing-pub
      configMap:
        name: lab-signing-pub
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
          sh "echo \"5 gstring: $CRED_PASS\""

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
    stage('mounted secret') {
      steps {
        container('helm') {
          sh 'ls -l /root/.kube'
          sh 'cat /root/.kube/config'
        }
      }
    }
    stage('managed file') {
      steps {
        configFileProvider([configFile(fileId: 'lab-answers', targetLocation: 'answers.yaml')]) {
          sh 'cat answers.yaml'
        }
        sh 'ls -l answers.yaml || echo GONE'
      }
    }
    stage('sign a chart') {
      steps {
        container('helm') {
          sh '''
            helm create labchart
            helm package --sign --key labsigner \
                 --keyring /tmp/signing-key.gpg \
                 --passphrase-file /tmp/passphrase \
                 labchart
            ls -l labchart-0.1.0.tgz*
            head -12 labchart-0.1.0.tgz.prov
            helm verify --keyring /pub/pubring.gpg labchart-0.1.0.tgz
          '''
        }
      }
    }
  }
}
