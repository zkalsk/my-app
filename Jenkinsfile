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
			deleteDir()
			checkout([$class: 'GitSCM',
                                  branches: [[name: '*/main']],
                                  extensions: [],
                                  userRemoteConfigs: [[credentialsId: 'jenkins', url: 'https://github.com/zkalsk/my-app.git']]
                        ])
			script {
				sh "sed -i 's/test:.*/test:v1.0.2/g' nginx.yaml"
				sh "git config user.name jenkins"
				sh "git config user.email admin@jenkins.com"
				withCredentials([usernameColonPassword(credentialsId: 'jenkins', usernameVariable: 'zkalsk', passwordVariable: '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')]) {
						sh "git add nginx.yaml"
                	                        sh "git commit -m '[update] image tag change'"
                        	                sh "git tag -a v1.0.2 -m 'v1.0.2'"
                               	                sh "git push origin main --tags"
				}
			}
		}
	}
   }
}

