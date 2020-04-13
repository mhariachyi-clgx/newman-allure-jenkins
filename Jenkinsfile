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
            REPORT_RICH_FILE_NAME = "${COLLECTION_NAME}_${ENVIRONMENT_NAME}_test_report.html"
            REPORT_COMPATIBLE_FILE_NAME = "simple.html"
            REPORT_JUNIT_NAME = "junit.xml"
            HTML_REPORT_TEMPLATE_PATH = "template-email-html.hbs"
    }
    options {
        ansiColor("xterm")
        timestamps()
        timeout(time: 1, unit: "HOURS")
        buildDiscarder(logRotator(numToKeepStr: '10'))
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
                    --reporter-htmlextra-export ${REPORT_FOLDER}/${REPORT_RICH_FILE_NAME} \
                    --reporter-html-export ${REPORT_FOLDER}/${REPORT_COMPATIBLE_FILE_NAME} \
                    --reporter-html-template ${HTML_REPORT_TEMPLATE_PATH} \
                    --reporter-junit-export ${REPORT_FOLDER}/${REPORT_JUNIT_NAME} \
                    -x"
                }
            }
            post {
                always {
                    echo "Saving reports..."
                    archiveArtifacts artifacts: "${REPORT_FOLDER}/**/*.*,allure-results/**/*.*", fingerprint: true
                    junit testResults: "${REPORT_FOLDER}/${REPORT_JUNIT_NAME}", allowEmptyResults: true
                    echo "junit passed"
                    allure results: [[path: "allure_results"]]
                    publishHTML([
                        allowMissing: true, 
                        alwaysLinkToLastBuild: false, 
                        keepAll: true, 
                        reportDir: "${REPORT_FOLDER}", 
                        reportFiles: "${REPORT_COMPATIBLE_FILE_NAME}", 
                        reportName: "Postman Report"
                    ])
                    script {
                        try {
                            sh "ls -LR reports"
                            def REPORT_HTML = readFile("${REPORT_FOLDER}/${REPORT_COMPATIBLE_FILE_NAME}").trim()
                            emailext([
                                recipientProviders: [[$class: "RequesterRecipientProvider"]],
                                subject: "Test Report ${COLLECTION_NAME} on ${ENVIRONMENT_NAME}",
                                body: REPORT_HTML,
                                mimeType: "text/html",
                                attachmentsPattern: "${REPORT_FOLDER}/${REPORT_RICH_FILE_NAME}"
                            ])
                        } catch (ex) {
                            echo ex.toString()
                        }
                    }
                }
            }
        }
    }
}
