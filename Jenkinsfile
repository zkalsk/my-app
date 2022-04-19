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
        stage('update k8s manifest') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']],
                        extensions: [],
                        userRemoteConfigs: [[credentialsId: 'jenkins', url: "https://github.com/zkalsk/my-app.git"]]])
                script {
                    sh "kustomize edit set image nginx=wlffjaso/test:${params.TAG}"
                    sh "git config user.name zkalsk"
                    sh "git config user.email wlffjaso@gmail.com"
                    withCredentials([usernamePassword(credentialsId: 'jenkins', passwordVariable: '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', usernameVariable: 'zkalsk')]) {
                        sh "git add . && git commit -m 'update image' && git push https://github.com/zkalsk/my-app.git HEAD:main || true"
                    }
                }
            }
        }
    }
}
