pipeline {
       agent any

       tools {
           dotnetsdk '.NET10'
       }

       triggers {
           githubPush()   // триггер на push
       }

       stages {
           stage('Checkout') {
               steps {
                   checkout scm
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
                  timeout(time: 5, unit: 'MINUTES') {
                             withCredentials([sshUserPrivateKey(credentialsId: 'c388a29f-ab16-44a6-ae08-fc2e14ed1f50',
                                                                keyFileVariable: 'SSH_KEY',
                                                                passphraseVariable: 'dog',
                                                                usernameVariable: 'root')]) {
                                 
                                    bat """
                                            echo SSH_KEY path: %SSH_KEY%
                                            whoami
                                            icacls "%SSH_KEY%" /inheritance:r /grant:r "Everyone:F"
                                            
                                            scp -v -o StrictHostKeyChecking=no -o ConnectTimeout=30 -i "%SSH_KEY%" -r ./publish/* root@38.244.216.252:/tmp/blazorapp/
                                        """
       
                                    bat """
                                            ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@38.244.216.252 
                                                   "sudo systemctl stop blazorapp || true && 
                                                   sudo rm -rf /var/www/testblazor/* && 
                                                   sudo mv /tmp/blazorapp/* /var/www/testblazor/ && 
                                                   sudo chown -R www-data:www-data /var/www/testblazor && 
                                                   sudo chmod -R 755 /var/www/testblazor && 
                                                   sudo systemctl start blazorapp && 
                                                   sudo systemctl status blazorapp --no-pager"
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
       }
   }
