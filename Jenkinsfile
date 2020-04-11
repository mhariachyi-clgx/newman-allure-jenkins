pipeline{
    agent{
        label 'corelogicpostman'
    }
    environment {
            REPORT_FOLDER = 'reports'
            COLLECTION_PATH = './AA_TEST.postman_collection.json'
            ENVIRONMENT_PATH = './search.postman_environment.json'
    }
    options {
        ansiColor('xterm')
        timestamps()
        timeout(time: 1, unit: 'HOURS')
    }
    stages {
        stage("Testing"){
            steps{
                script{
                    currentBuild.description = "${env.COLLECTION_PATH} tests on ${env.ENVIRONMENT_PATH}"
                    sh 'npm install newman'
                    sh 'npm install newman-reporter-allure'
                    sh 'npm install newman-reporter-htmlextra'
                    sh "node_modules/newman/bin/newman.js run ${COLLECTION_PATH} \
                    -e ${ENVIRONMENT_PATH} \
                    -r cli,junit,allure,htmlextra \
                    --reporter-htmlextra-export ${REPORT_FOLDER}/extra.html \
                    --reporter-junit-export ${REPORT_FOLDER}/junit.xml \
                    -x"
                }
            }
            post {
                always {
                    echo 'Saving reports...'
                    archiveArtifacts artifacts: "${REPORT_FOLDER}/**/*.*,allure-results/**/*.*", fingerprint: true
                    junit testResults: "${REPORT_FOLDER}/junit.xml", allowEmptyResults: true
                    echo 'junit passed'
                    allure results: [[path: 'allure_results']]
                }
            }
        }
    }
    post {
        always {
             script {
                try {
                    //sh 'echo REPORT_HTML=$(cat reports/extra.html)'
                    sh 'ls -LR reports'
                    def REPORT_HTML = readFile('${REPORT_FOLDER}/extra.html').trim()
                    emailext([body: REPORT_HTML, mimeType: 'text/html', attachmentsPattern: '${REPORT_FOLDER}/extra.html', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'WhitneyVectorRules Smoke Test Report'])
                } catch (ex) {
                    echo ex.toString()
                }
            }
        }
    }
}
