# testrail-promise
TestRail wrapper using promises

[!["npm badge"](https://nodei.co/npm/testrail-promise.png)](https://www.npmjs.com/package/testrail-promise)


Promise implementation of the TestRail API that returns JSON

http://docs.gurock.com/testrail-api2/start


## Setup
```
var TestRail = require("testrail-promise");
var tr = new TestRail("<url>", "<user email>", "<password/apikey>");
```

## Simple Use Cases

```
//Getting all Projects
tr.getProjects()
    .then(function(projects) { ... })
    .catch(function(err) { ... });

//Getting a specific project
tr.getProject({"project_id":6})
    .then(function(project) { ... })
    .catch(function(err) { ... });

//Getting test cases
tr.getCases({"project_id":"6", "suite_id":"6", "section_id":"173"})
    .then(function(case) { ... })
    .catch(function(err) { ... });    
```
## Automation Use Case

If automated test (Protractor/Karma/...) results are wanted in TestRail use this function.
```
var obj = {
    "project_name":"<project name>",
    "plan_name":"<test plan>",
    "section_name":"<section/test case folder>",
    "title":"<title of test case>",
    "status_name":"<passed/failed/retest/blocked>"
};
tr.ifNeededCreateThenAddResultForCase(obj)
    .then(function (result) { ... })
    .catch(function (err) { ... });
```
This will create a test case under the specific section if one does not exist, then update the result to the most recent run of a test plan.
It will query using the API to get IDs hence names can be used as long as they are **unique**. The only required information is: project (`project_id`/`project_name`), plan(`plan_id`/`plan_name`),section(`section_id`/`section_name`), `title`(can use `case_id` if case already exists) and status(`status_id`/`status_name`).

Jasmine considers showing results within a tests a leak, so we have to use a [custom reporter](http://jasmine.github.io/2.1/custom_reporter.html) to get results. Usually, we would send the results in a `specDone` function, but it is not asynchronous therefore I suggest adding an object onto the `jasmine` var. Here is an example of how to report tests in Protractor:

```
resultLeaker = {
  suiteStarted: function(result){ jasmine.results = {suite:result}; },
  specStarted: function(result){ jasmine.results.spec = result; }
};
jasmine.getEnv().addReporter(resultLeaker);

describe('TestRail Reporter', function(){
    it('Passing', function(){
        expect(true).toBeTruthy();
    });
    it('Failing', function(){
        expect(false).toBeTruthy();
    });

    var TestRail = require("testrail-promise");
    var tr = new TestRail("<url>", "<user email>", "<password/apikey>");

    afterEach(function(done){
        var obj = {
            "project_name":"<project name>",
            "plan_name":"<test plan>",
            "section_name":"<section/test case folder>",
            "title":jasmine.results.spec.fullName,
            "status_name":(jasmine.results.spec.failedExpectations.length === 0 ? "passed" : "failed")
        };
        tr.ifNeededCreateThenAddResultForCase(obj).finally(function(){
            done();
        });
    });
});
```

## Additional Information

All functions that need input take just one object. It uses what is needed for the API URI then sends the object to the API. This means you should be able to use any field available in the API documentation.
```
var object = {
    "section_id":173,
    "title":"testrail-promise",
    "type_id":3,
    "project_id":6,
    "estimate":1,
    "milestone_id":3,
    "refs":""
};
tr.addCase(object);
```

If for some reason the cert on TestRail is not trusted, this function can be called to ignore that error.
```
tr.allowUntrustedCertificate();
```

If you want to get errors on non-200 responses change to simple requests.
```
tr.simpleRequests();
```
