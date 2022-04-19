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
	        git credentialsId: 'github-credential',
                    url: 'https://github.com/zkalsk/k8s-manifest.git',
                    branch: 'main'
                script {
                    sh "sed -i 's/test:.*/test:${params.TAG}/g' nginx.yaml"
                    sh "git init"
	  	    sh "git add ."
		    sh "git commit -m 'update image'"
                    withCredentials([gitUsernamePassword(credentialsId: 'github-credential',  gitToolName: 'Default')]) {
			sh "git push origin main"
                    }
                }
            }
        }
    }
}
