pipeline{
    agent{
        label "corelogicpostman"
    }
    environment {
            REPORT_FOLDER = "reports"
            COLLECTION_NAME = "DemoSuite"
            ENVIRONMENT_NAME = "demo_env"
            COLLECTION_PATH = "./${COLLECTION_NAME}.postman_collection.json"
            ENVIRONMENT_PATH = "./${ENVIRONMENT_NAME}.postman_environment.json"
            REPORT_FILE_NAME = "${COLLECTION_NAME}_${ENVIRONMENT_NAME}_test_report.html"
    }
    options {
        ansiColor("xterm")
        timestamps()
        timeout(time: 1, unit: "HOURS")
    }
    stages {
        stage("Testing"){
            steps{
                script{
                    currentBuild.description = "${env.COLLECTION_NAME} tests on ${env.ENVIRONMENT_NAME}"
                    sh "npm install newman"
                    sh "npm install newman-reporter-allure"
                    sh "npm install newman-reporter-html"
                    sh "npm install newman-reporter-htmlextra"
                    sh "node_modules/newman/bin/newman.js run ${COLLECTION_PATH} \
                    -e ${ENVIRONMENT_PATH} \
                    -r cli,junit,allure,html,htmlextra \
                    --reporter-htmlextra-export ${REPORT_FOLDER}/${REPORT_FILE_NAME} \
                    --reporter-html-export ${REPORT_FOLDER}/simple.html \
                    --reporter-html-template template-simple.hbs \
                    --reporter-junit-export ${REPORT_FOLDER}/junit.xml \
                    -x"
                }
            }
            post {
                always {
                    echo "Saving reports..."
                    archiveArtifacts artifacts: "${REPORT_FOLDER}/**/*.*,allure-results/**/*.*", fingerprint: true
                    junit testResults: "${REPORT_FOLDER}/junit.xml", allowEmptyResults: true
                    echo "junit passed"
                    allure results: [[path: "allure_results"]]
                }
            }
        }
    }
    post {
        always {
             script {
                try {
                    sh "ls -LR reports"
                    def REPORT_HTML = readFile("${REPORT_FOLDER}/simple.html").trim()
                    emailext([
                        recipientProviders: [[$class: "RequesterRecipientProvider"]],
                        subject: "Test Report ${COLLECTION_NAME} on ${ENVIRONMENT_NAME}",
                        body: REPORT_HTML,
                        mimeType: "text/html",
                        attachmentsPattern: "${REPORT_FOLDER}/${REPORT_FILE_NAME}"
                    ])
                } catch (ex) {
                    echo ex.toString()
                }
            }
        }
    }
}
