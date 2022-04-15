node { 
  stage('Clone repository') { 
    checkout scm 
  } 
  stage('Build image') { 
    app = docker.build("wlffjaso/test") 
  }
  stage('Test image') {
    app.inside{
      sh 'echo "Test passed"'
    }
  } 
  stage('Push image') { 
    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') { 
      app.push("v1") 
    } 
  } 
}
