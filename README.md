# newman-allure-jenkins
Jenkins pipeline configuration to run Postman tests with Allure reporter
```shell script
node_modules/newman/bin/newman.js run ./DemoSuite.postman_collection.json \
-e ./search.postman_environment.json \
-r cli,junit,allure,html \
--reporter-html-export reports/extra.html \
--reporter-junit-export reports/junit.xml
```

