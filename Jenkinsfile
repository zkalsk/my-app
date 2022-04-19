pipeline {
    agent any
    parameters {
        gitParameter name: 'TAG',
        type: 'PT_TAG',
        defaultValue: 'v1.2'
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
        stage('update k8s manifest') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[credentialsId: 'jenkins', url: "https://github.com/zkalsk/my-app.git"]]])
                script {
                    sh "sed -i 's/test:.*/test:${params.TAG}/g' nginx.yaml"
                    sh "git config user.name zkalsk"
                    sh "git config user.email wlffjaso@gmail.com"
                    sshagent(['jenkins']) {
                        sh "git add . && git commit -m 'update image' && git push origin main --tags"
                    }
                }
            }
        }
    }
}
