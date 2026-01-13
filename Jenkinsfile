pipeline {
    agent any

    environment {
        DEPLOY_HOST = '38.244.216.252'
        DEPLOY_USER = 'root'
        DEPLOY_PATH = '/var/www/testblazor'
        TEMP_PATH = '/tmp/blazorapp'
    }

    tools {
        // Проверьте точное имя .NET SDK в Jenkins -> Global Tool Configuration
        dotnetsdk 'NET10'  // Или 'dotnet-10.0' - зависит от настройки
    }

    triggers {
        pollSCM('H/5 * * * *')  // Проверять каждые 5 минут
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/RamilRakaev/TestBlazor.git',
                        credentialsId: 'c388a29f-ab16-44a6-ae08-fc2e14ed1f50'
                    ]]
                ])
            }
        }
        
        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }
        
        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release --no-restore'
            }
        }
        
        stage('Publish') {
            steps {
                bat 'dotnet publish TestBlazor/TestBlazor.csproj --configuration Release --output ./publish --no-build'
            }
        }
        
        stage('Deploy to VDS') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'c388a29f-ab16-44a6-ae08-fc2e14ed1f50',
                        keyFileVariable: 'SSH_KEY',
                        passphraseVariable: '',  // Если у ключа нет пароля
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    script {
                        // Сохраняем путь к ключу в переменную с правильными слешами
                        def sshKeyPath = "${env.SSH_KEY}".replace('\\', '/')
                        
                        // Подготовка SSH ключа - УБРАТЬ изменения прав (это вызывает проблемы)
                        bat """
                            echo Using SSH key at: ${sshKeyPath}
                            dir "${env.SSH_KEY}"
                        """
                        
                        // 1. Копируем файлы на сервер
                        bat """
                            scp -o StrictHostKeyChecking=no -i "${env.SSH_KEY}" -r ./publish/* ${env.SSH_USER}@${env.DEPLOY_HOST}:${env.TEMP_PATH}/
                        """
                        
                        // 2. Выполняем команды на сервере
                        bat """
                            ssh -o StrictHostKeyChecking=no -i "${env.SSH_KEY}" ${env.SSH_USER}@${env.DEPLOY_HOST} "
                                sudo systemctl stop blazorapp 2>/dev/null || true
                                sudo rm -rf ${env.DEPLOY_PATH}/*
                                sudo mv ${env.TEMP_PATH}/* ${env.DEPLOY_PATH}/
                                sudo chown -R www-data:www-data ${env.DEPLOY_PATH}
                                sudo chmod -R 755 ${env.DEPLOY_PATH}
                                sudo systemctl start blazorapp
                                echo 'Deployment completed successfully!'
                                sudo systemctl status blazorapp --no-pager
                            "
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'publish/**', allowEmptyArchive: true
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            // Можно добавить уведомления
        }
    }
}
