pipeline {
  agent {
    kubernetes {
      label 'master'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
namespace: sybrenbolandit
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker:18.09
    command:
    - cat
    tty: true
  - name: tools
    image: argoproj/argo-cd-ci-builder:v0.13.1
    command:
    - cat
    tty: true
"""
    }
  }
  stages {

    stage('Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
      }
      steps {
        container('docker') {
          // Build new image
          sh "until docker ps; do sleep 3; done && docker build -t atul7cloudyuga/argocd-demo:${env.GIT_COMMIT} ."
        
        }
      }
    }
    
    stage('Deploy E2E') {
      environment {
        GIT_CREDS = credentials('git')
      }
      steps {
        container('tools') {
          sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/atul7cloudyuga/argocd-demo-deploy.git"
          sh "git config --global user.email 'atul@ccloudyuga.guru'"

          dir("argocd-demo-deploy") {
            sh "cd ./e2e && kustomize edit set image atul7cloudyuga/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Publish new version' && git push || echo 'no changes'"
          }
        }
      }
    }

  
    }
  }
}
