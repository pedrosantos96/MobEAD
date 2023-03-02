pipeline {  
    agent any 
    stages { 
        stage('Remove container antigo'){ 
            steps{
                script {
                    try {
                        sh 'docker rm -f mobead_image_build-dev'
                    } catch (Exception e) {
                        sh "echo $e"
                    }
                }
            }    
        }
        stage('Subindo container de desenvolvimento') {
            steps{
                script {
                    try {
                        sh 'docker run -d -p 81:80 --name=mobead_image_build-dev $image'
                    } catch (Exception e) {
                        slackSend (color: 'error', message: "[ FALHA ] Não foi possível subir o container - ${BUILD_URL} em ${currentBuild.duration}s", tokenCredentialId: 'Slack-token')
                        sh "echo $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro')
                    }
                }
            }
        }
        stage('Notificando o usuario') {
            steps{
                slackSend (color: 'good', message: '[ Sucesso ] O novo build esta disponivel em: http://192.168.15.5:81/', tokenCredentialId: 'Slack-token')
            }
        }
        stage ('Fazer o deploy em producao?') {
            steps {
                slackSend (color: 'warning', message: "Para aplicar a mudança em produção, acesse [Janela de 10 minutos]: ${JOB_URL}", tokenCredentialId: 'Slack-token')
                timeout(time: 10, unit: 'MINUTES') {
                    input(id: "DeployGate", message: "Deploy em produção?", ok: 'Deploy')
                }
            }
        }
        stage (deploy) {
            steps {
                script {
                    try {
                        build job: 'Produção-pipeline-Unyleya-Pedro-Fagundes', parameters: [[$class: 'StringParameterValue', name: 'image', value: image]]
                    } catch (Exception e) {
                        slackSend (color: 'error', message: "[ FALHA ] Não foi possível subir o container em producao - ${BUILD_URL}", tokenCredentialId: 'Slack-token')
                        sh "echo $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro')
                    }
                }
            }
        }
    }
}