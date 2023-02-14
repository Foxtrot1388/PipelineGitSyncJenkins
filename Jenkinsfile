#!groovy
// Check properties
properties([disableConcurrentBuilds()])

pipeline {

    agent { 
        label 'GitSync'
        }

    environment {
        SRCPATH=credentials('SrcPath')
        BOTID=credentials('BotID')
        CHATID=credentials('ChatID')
        TEXTSUCCESS="SUCCESS"
        TEXTUNSUCCESSFUL="UNSUCCESSFUL"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()
        timeout(time: 2, unit: 'HOURS')
    }

    stages {

        // выгружаем данные из хранилища
        stage('Unloading from storage') {
            environment {
                USERSTORAGE=credentials('UserStorage1C')
                PASSSTORAGE=credentials('PassStorage1C')
                STORAGEPATH=credentials('StoragePath')
            }
            steps {
                bat "@chcp 1251 && set GITSYNC_WORKDIR=${SRCPATH} && set GITSYNC_STORAGE_PATH=${STORAGEPATH} && set GITSYNC_VERBOSE=true && set GITSYNC_V8VERSION=8.3.19.1331 && set GITSYNC_V8_PATH=C:\\1C\\8.3.19.1331\\bin\\1cv8.exe && gitsync sync -u ${USERSTORAGE} -p ${PASSSTORAGE}"
            }
        }

        // компилируем конфигурацию
        stage('Compile CF') {
            environment {
                CFPATH="${WORKSPACE}\\1Cv8.cf"
            }
            steps {
                bat "@chcp 1251 && @call vrunner compile --v8version 8.3.19.1331 --src ${SRCPATH} --out ${CFPATH}"    
            }
            post {
                always {
                    archiveArtifacts artifacts: '1Cv8.cf', onlyIfSuccessful: true
                }
            }
        }

    }

    post {
        success { 
            bat "oscript ${WORKSPACE}/NotifyTelegram.os ${BOTID} ${CHATID} ${TEXTSUCCESS}"    
        }
        unsuccessful { 
            bat "oscript ${WORKSPACE}/NotifyTelegram.os ${BOTID} ${CHATID} ${TEXTUNSUCCESSFUL}"   
        }
    }

}