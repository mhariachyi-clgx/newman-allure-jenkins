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
                    emailext([body: "Hi", mimeType: 'text/html', attachmentsPattern: 'reports/extra.html', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'WhitneyVectorRules Smoke Test Report'])
                    sh 'REPORT_HTML=$(cat reports/extra.html)'
                    allure results: [[path: 'allure_results']]
                    junit testResults: 'reports/ST/PropertyImage.xml', allowEmptyResults: true
                    echo 'junit passed'
                }
            }
        }
    }
}
