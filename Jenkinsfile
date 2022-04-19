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
		git credentialsId: 'github-credential',
		    url: 'https://github.com/zkalsk/k8s-manifest.git',
		    branch: 'main'
		sh "sed -i 's/test:.*/test:${params.TAG}/g' nginx.yaml"
		sh "git add nginx.yaml"
		sh "git commit -m 'update image version'"
		sshagent(['jenkins']) {
			sh "git remote set-url origin git@github.com:zkalsk/k8s-manifest.git"
			sh "git push -u origin main"
		}
            }
        }
    }
}
