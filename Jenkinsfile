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
                      withCredentials([sshUserPrivateKey(credentialsId: 'vds-ssh-key',
                                                         keyFileVariable: 'SSH_KEY',
                                                         usernameVariable: 'SSH_USER')]) {
                          bat """
                              scp -i %SSH_KEY% -r ./publish/* %SSH_USER%@38.244.216.252:/tmp/blazorapp/
                              ssh -i %SSH_KEY% %SSH_USER%@38.244.216.252 ^
                                  "sudo systemctl stop blazorapp || true && ^
                                   sudo rm -rf /var/www/testblazor/* && ^
                                   sudo mv /tmp/blazorapp/* /var/www/testblazor/ && ^
                                   sudo chown -R www-data:www-data /var/www/testblazor && ^
                                   sudo chmod -R 755 /var/www/testblazor && ^
                                   sudo systemctl start blazorapp && ^
                                   sudo systemctl status blazorapp --no-pager"
                          """
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
