# newman-allure-jenkins
Jenkins pipeline configuration to run Postman tests with Allure reporter
```shell script
node_modules/newman/bin/newman.js run DemoApp/DemoSuite.postman_collection.json \
-e DemoApp/demo_env.postman_environment.json \
-d DemoApp/test_data.csv \
-r cli,junit,allure,html,htmlextra \
--reporter-htmlextra-export reports/extra.html \
--reporter-html-export reports/simple.html \
--reporter-html-template template-email-html.hbs \
--reporter-junit-export reports/junit.xml \
-x
```

