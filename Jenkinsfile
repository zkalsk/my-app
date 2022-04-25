pipeline {
    agent any
    parameters {
        gitParameter name: 'TAG',
        type: 'PT_TAG',
        defaultValue: 'v1.3'
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
                        userRemoteConfigs: [[url: 'https://github.com/zkalsk/helm-chart.git']]
                ])
                script {
		    def value = readYaml file: "values.yaml"
                    value.image.tag = "${params.TAG}"
		    sh "rm -f values.yaml"
		    writeYaml file: "values.yaml", data: value
                    sh 'cat values.yaml'
				
   		    def chart = readYaml file: "Chart.yaml"
                    chart.appVersion = "${params.TAG}"
		    chart.version = "${params.TAG}"
		    sh "rm -f Chart.yaml"
		    writeYaml file: "Chart.yaml", data: chart
                    sh 'cat Chart.yaml'
				
		    sh "helm repo index ."
		    sh "helm package ."
				
		    sh "git config --global user.name 'zkalsk'"
	            sh "git config --global user.email 'wlffjaso@gmail.com'"
	            sh "git add ."
	            sh "git commit -m 'update image version ${params.TAG}'"
                    sh "git remote -v"
                    sh "git status"
                    withCredentials([gitUsernamePassword(credentialsId: 'github-credential',  gitToolName: 'git-tool')]) {
	                sh "git push origin HEAD:main"
		    }
                }
            }
        }
    }
}
