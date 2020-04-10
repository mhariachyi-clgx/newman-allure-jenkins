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
                    sh "sudo node_modules/newman/bin/newman.js run ./AA_TEST.postman_collection.json \
                    -e ./search.postman_environment.json \
                    -r cli,junit,allure,htmlextra \
                    --reporter-htmlextra-export reports/extra.html \
                    --reporter-junit-export reports/${params.Suite}.xml"
                }
            }
            post {
                always {
                    echo 'Saving reports...'
                    archiveArtifacts artifacts: 'reports/**/*.*,allure-results/**/*.*', fingerprint: true
                    echo 'archiveArtifacts passed'
                    sh 'echo REPORT_HTML=$(cat reports/extra.html)'
                    junit testResults: 'reports/ST/PropertyImage.xml', allowEmptyResults: true
                    echo 'junit passed'
                }
            }
        }
    }
    post {
        always {
            allure results: [[path: 'allure_results']]
            emailext([body: $REPORT_HTML, mimeType: 'text/html', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'WhitneyVectorRules Smoke Test Report', to: 'mhariachyi@corelogic.com'])
            publishHTML([reportDir: 'reports', reportFiles: 'extra.html', reportName: 'HTML Report', allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true])
        }
    }
}
