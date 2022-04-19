pipeline {
    agent any
    parameters {
        gitParameter name: 'TAG',
                     type: 'PT_TAG',
                     defaultValue: 'v1.0'
    }
    stages {
        stage('checkout code/scm') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: "${params.TAG}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          gitTool: 'Default',
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: 'https://github.com/zkalsk/my-app.git']]
                        ])
            }
        }
        stage('build') {
                steps {
                        script {
                                dockerImage = docker.build("wlffjaso/test")
                        }
                }
        }
        stage('push image') {
                steps {
                        script {
                                docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                                        dockerImage.push("${params.TAG}")
                                }
                        }
                }
        }
    }
}
