# newman-allure-jenkins
Jenkins pipeline configuration to run Postman tests with Allure reporter
```shell script
node_modules/newman/bin/newman.js run ./DemoSuite.postman_collection.json \
-e ./demo_env.postman_environment.json \
-r cli,junit,allure,html,htmlextra \
--reporter-htmlextra-export reports/extra.html \
--reporter-html-export reports/simple.html \
--reporter-html-template template-simple.hbs \
--reporter-junit-export reports/junit.xml \
-x
```

