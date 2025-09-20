pipeline {
    agent any
    parameters { 
        string(name: 'GIT_CREDENTIALS_ID', defaultValue: 'github-ssh', description: 'Git SSH Key Credential ID')
        string(name: 'BRANCH_NAME', defaultValue: 'main')
    }
    triggers {
        githubPush() 
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    userRemoteConfigs: [[
                        url: 'git@github.com:miairan/jenkins-vue-demo.git',
                        credentialsId: "${params.GIT_CREDENTIALS_ID}"
                    ]],
                    branches: [[name: "*/${params.BRANCH_NAME}"]],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: [],
                    extensions: []
                ])
                
            }
        }
        stage('Debug') {
            steps {
                echo "🧪 Triggered at: ${new Date()}"
                echo "🧪 Triggered by WebHook"
            }
        }
        stage('Prepare') {
            steps {
                script {
                    def commitHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim().replaceAll("[^a-zA-Z0-9]", "") 
                    def imageName = "jenkins-vue-demo:${commitHash}"
                    writeFile file: '.image_name', text: imageName
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    def imageName = readFile('.image_name').trim()
                    echo "🛠️ 构建镜像：${imageName}"
                    sh "docker build --load -t $imageName ."
                }
            }
        }
        stage('Docker Run') {
            steps {
                script {
                    def imageName = readFile('.image_name').trim()
                    def basePort = 8088
                    def hostPort = basePort
                    while (true) {
                        def inUse = sh(script: "lsof -i :${hostPort}", returnStatus: true)
                        if (inUse != 0) {
                            break // 找到空闲端口
                        }
                        hostPort++
                    }

                    echo "✅ 使用端口 ${hostPort}"
                    sh """#!/bin/bash
                        echo "🧹 停止并删除旧容器（如果存在）"
                        docker stop jenkins-vue-demo || true
                        docker rm jenkins-vue-demo || true
                        echo "🚀 启动新容器${imageName}"
                        docker run -d -p ${hostPort}:80 --name jenkins-vue-demo ${imageName}
                    """
                }
            }
        }
    }
    post {
        always {
            echo "构建完成"
        }
    }
}
