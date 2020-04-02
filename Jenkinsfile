pipeline{
    agent{
        label 'corelogicpostman'
    }

    post {
        always {
            allure report: 'allure_reports', results: [[path: 'allure_results']]
        }
    }
}
