pipeline{
    agent{
        label 'corelogicpostman'
    }
    options {
        ansiColor('xterm')
        timestamps()
    }
    stages {
        stage("Testing"){
            steps{
                script{
                    currentBuild.description = "${params.Suite} tests on ${params.Environment}"
                    sh 'npm install newman'
                    sh 'npm install newman-reporter-allure'
                    sh 'npm install newman-reporter-htmlextra'
                    sh "node_modules/newman/bin/newman.js run ./AA_TEST.postman_collection.json \
                    -e ./search.postman_environment.json \
                    -r cli,junit,allure,html \
                    --reporter-html-export reports/extra.html \
                    --reporter-junit-export reports/junit.xml"
                }
            }
            post {
                always {
                    echo 'Saving reports...'
                    archiveArtifacts artifacts: 'reports/**/*.*,allure-results/**/*.*', fingerprint: true
                    echo 'archiveArtifacts passed'
                    sh 'echo REPORT_HTML=$(cat reports/extra.html)'
                    emailext([body: "$REPORT_HTML", mimeType: 'text/html', attachmentsPattern: 'reports/extra.html', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'WhitneyVectorRules Smoke Test Report'])
                    allure results: [[path: 'allure_results']]
                    junit testResults: 'reports/junit.xml', allowEmptyResults: true
                    echo 'junit passed'
                }
            }
        }
    }
}
