pipeline{
    agent{
        label "corelogicpostman"
    }
    environment {
    		Application = "DemoSuite"
            Environment = "demo_env"
            COLLECTION_NAME = getFileName(Application)
            APP_FOLDER = getFoldersPath(Application)
            ENVIRONMENT_NAME = Environment
            COLLECTION_PATH = "./${APP_FOLDER}${COLLECTION_NAME}.postman_collection.json"
            ENVIRONMENT_PATH = "./${APP_FOLDER}${ENVIRONMENT_NAME}.postman_environment.json"
            REPORT_FOLDER = "${APP_FOLDER}reports/"
            REPORT_RICH_FILE_NAME = "${COLLECTION_NAME}_${ENVIRONMENT_NAME}_test_report_extra.html"
            REPORT_COMPATIBLE_FILE_NAME = "${COLLECTION_NAME}_${ENVIRONMENT_NAME}_test_report.html"
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
                script {
                    currentBuild.description = "${env.COLLECTION_NAME} tests on ${env.ENVIRONMENT_NAME}"
                    sh "npm install newman"
                    sh "npm install newman-reporter-allure"
                    sh "npm install newman-reporter-html"
                    sh "npm install newman-reporter-htmlextra"
                    //TODO: change 'node_modules/newman/bin/newman.js' to 'newman' when newman-reporter-allure installed globally
                    sh "node_modules/newman/bin/newman.js run ${COLLECTION_PATH} \
                    -e ${ENVIRONMENT_PATH} \
                    -r cli,junit,allure,html,htmlextra \
                    --reporter-junit-export ${REPORT_FOLDER}/${REPORT_JUNIT_NAME} \
                    --reporter-html-export ${REPORT_FOLDER}/${REPORT_COMPATIBLE_FILE_NAME} \
                    --reporter-html-template ${HTML_REPORT_TEMPLATE_PATH} \
                    --reporter-htmlextra-export ${REPORT_FOLDER}/${REPORT_RICH_FILE_NAME} \
                    -x"
                }
            }
            post {
                always {
                    echo "Saving reports..."
                    archiveArtifacts artifacts: "${REPORT_FOLDER}/**/*.*,allure-results/**/*.*", fingerprint: true
                    echo "archiveArtifacts finished"
                    junit testResults: "${REPORT_FOLDER}/${REPORT_JUNIT_NAME}", allowEmptyResults: true
                    echo "junit finished"
                    allure results: [[path: "allure_results"]]
                    echo "allure finished"
                    publishHTML([
                        allowMissing: true, 
                        alwaysLinkToLastBuild: false, 
                        keepAll: true, 
                        reportDir: "${REPORT_FOLDER}", 
                        reportFiles: "${REPORT_COMPATIBLE_FILE_NAME}", 
                        reportName: "Postman Report"
                    ])
                    echo "publishHTML finished"
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
                            echo "emailext finished"
                        } catch (ex) {
                            echo ex.toString()
                        }
                    }
                    echo "All the reports finished"
                }
            }
        }
    }
}

def getFileName(String path) {
	return path.substring(path.lastIndexOf('/') + 1)
}

def getFoldersPath(String path) {
	def lastIndex = path.lastIndexOf('/')
	return path.substring(0, lastIndex < 0 ? lastIndex + 1 : lastIndex)
}