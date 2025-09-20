pipeline {
    agent any

    triggers {
        githubPush()
    }

    stages {
        stage('Build') {
            steps {
                echo "✅ Webhook Triggered - 构建成功！"
            }
        }
    }
}