pipeline {
    agent any
    parameters {
        gitParameter name: 'TAG',
        type: 'PT_TAG',
        defaultValue: 'v1.1'
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
                    dockerImage = docker.build("wlffjaso/test-cicd")
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
                        branches: [[name: "*/main"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        gitTool: 'Default',
                        submoduleCfg: [],
                        userRemoteConfigs: [[url: 'https://github.com/zkalsk/k8s-manifest.git']]
                    ])
                script {
                    sh "sed -i 's/test:.*/test:${params.TAG}/g' nginx.yaml"
                    sh "git config user.name zkalsk"
                    sh "git config user.email wlffjaso@gmail.com"
                    withCredentials([usernamePassword(credentialsId: 'github-credential', passwordVarible: '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', usernameVariable: 'zkalsk')]) {
                        sh "git add nginx.yaml && git commit -m 'update image' && git push origin main"
                    }
                }
            }
        }
    }
}
