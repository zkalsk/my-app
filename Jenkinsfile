pipeline {
    agent any
    parameters {
        gitParameter name: 'TAG',
                     type: 'PT_TAG',
                     defaultValue: 'v1.0.0'
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
	stage('k8s manifest update') {
		steps {
			sh "sed -i 's/test:.*/test:v1.0.1/g' nginx.yaml"
			sh "git add nginx.yaml"
			sh "git commit -m '[update] image tag change'"
			sh "git tag -a v1.0.1 -m 'v1.0.1'"
			sh "git push origin main --tags"
		}
	}
   }
}

