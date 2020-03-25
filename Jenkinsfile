imageTag="test"
pipeline {
  environment {
      Namespace = "default"
      ImageName = "saifrahm/hello-plain"
      Creds = "615cfb48-e44f-4b51-85ca-5c6a8ab22b8e"
  }
  agent any
  stages {
    stage('Checkout') {
      steps {
        sh 'printenv'
        sh "git rev-parse --short HEAD > .git/commit-id"
         script {
             imageTag= readFile('.git/commit-id').trim()
      }
     }
    }
    stage('Build and push image with Container Builder') {
      steps {
        withDockerRegistry([credentialsId: "${Creds}", url: 'https://index.docker.io/v1/']) {
        sh "docker build -t ${ImageName}:${imageTag} ."
        sh "docker push ${ImageName}:${imageTag}"
        }
      }
    }
    stage('Deploy Canary') {
      // Canary branch
      when { branch 'canary' }
      steps {
         sh "/usr/local/bin/helm --kubeconfig /var/cluster150/admin.conf upgrade can-app /var/demochart --set canaryImage.tag=${imageTag} --set canaryIngress.enabled=true  --install --namespace ${Namespace} --wait"
        }
      }
    stage('Deploy Production') {
      // Production branch
      when { branch 'master' }
      steps{
        sh "/usr/local/bin/helm --kubeconfig /var/cluster150/admin.conf upgrade can-app /var/demochart --set image.tag=${imageTag} --set canaryIngress.enabled=false --install --namespace ${Namespace} --wait"
      }
    }
    }
  }
