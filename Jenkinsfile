pipeline {
    agent any

    environment {
        GCP_PROJECT_ID = 'seu-projeto-no-gcp'
        GCP_ZONE = 'sua-zona-no-gcp'
        GCP_INSTANCE_GROUP_PROD = 'seu-instance-group-de-prod'
        MANAGER_EMAIL = 'email-do-gerente@empresa.com'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Verificar Release') {
            steps {
                script {
                    def isRelease = env.GITHUB_REF.startsWith('refs/tags/')
                    
                    if (isRelease) {
                        echo "Este push foi feito através de uma release."
                        // Faça o que precisa ser feito para uma release
                    } else {
                        echo "Este push não foi feito através de uma release."
                        // Faça o que precisa ser feito para outros tipos de push
                    }
                }
            }
        }        

        stage('Deploy to Prod') {
            // when {
            //     changeset "**"
            // }
            steps {
                script {
                    // Implementar lógica de deploy para instâncias de produção com tags
                    def tag = deployToProd()
                    // Aguardar aprovação do gerente
                    input message: "Aprovar deploy para produção?", parameters: [
                        string(defaultValue: '', description: 'Comentários', name: 'comments')
                    ]
                    // Envie um link para o gerente para autorizar o deploy
                    sendApprovalEmail(tag)
                }
            }
        }
    }

    post {
        failure {
            script {
                // Execute o estágio de rollback em caso de falha
                rollback()
            }
        }
    }
}

def deployToProd() {
    // Gerar uma tag única com timestamp para a versão
    def tag = "deploy-" + sh(script: 'date +"%Y%m%d%H%M%S"', returnStdout: true).trim()
    
    
    // Implementar lógica de deploy para instâncias de produção com tags
    // Use gcloud CLI ou Google Cloud Jenkins Plugin para fazer o deploy
    
    // Exemplo para uma aplicação PHP
    sshagent(credentials: ['6703ec14-cc0a-4948-9a58-7c799c4f2e14'], ignoreMissing: true) {
        // some block
        def tagAtual = "ssh -o StrictHostKeyChecking=no usuario@10.11.19.30 'git tag -l --sort=-creatordate | head -n 2 | tac | head -n 1'"
        echo "Tag Atual : ${tagAtual}"
        sh "ssh -o StrictHostKeyChecking=no usuario@10.11.19.30 'cd /home/usuario/valida-apagar && git fetch --tags && git checkout ${tag} && systemctl restart apache2'"
    }
    
    
    return tag
}

def sendApprovalEmail(tag) {
    // Lógica para enviar um e-mail para o gerente com um link para aprovação do deploy
    // Use o plugin de e-mail do Jenkins ou outra solução de envio de e-mail
    def emailSubject = "Aprovação para Deploy - Tag ${tag}"
    def emailBody = "Por favor, clique no link abaixo para aprovar o deploy:\n\n[Link de Aprovação](http://seu-sistema-de-aprovacao/${tag})"
    
    emailext subject: emailSubject, body: emailBody, to: MANAGER_EMAIL
}

def rollback() {
    // Implemente a lógica de rollback para a última tag com deploy bem-sucedido
    // Use gcloud CLI ou Google Cloud Jenkins Plugin para fazer o rollback
    def lastSuccessfulTag = sh(script: 'git describe --tags --abbrev=0 --match "deploy-success-*"', returnStdout: true).trim()
    
    echo "Realizando rollback para a última versão com deploy bem-sucedido: ${lastSuccessfulTag}"
    
    // Exemplo: sh "gcloud ... --tags=${lastSuccessfulTag}"
}
