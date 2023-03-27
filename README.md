wiremock-loader
===============

Wiremock-Loader is designed for environments where a wiremock service is
available, but cannot be initialized via the filesystem. (Gitlab CI Services)

The intention is to read the `mappings` / `__files` from the local filesystem,
and inject the stub definitions via the `__admin` interface.

Usage
-----

The system is intended to be run from a system that has yarn / node installed,
such as the `srsbiznas/sbt_yarn_ci` docker image.

    yarn start -m /path/to/mocks -h mock-host -p 12345

> Note that the default value for the hostname is `localhost` and the default
> port is 8080.


Wiremock Stateful behavior 
-----
https://wiremock.org/docs/stateful-behaviour/

The following changes were made to support scenarios:

1. When handling stubs with scenarios, the `persistent`, `priority`, `scenarioName`, `requiredScenarioState`, and `newScenarioState` properties are added to the post body sent to WireMock.
````javascript
if (mappingData.scenarioName || mappingData.requiredScenarioState || mappingData.newScenarioState) {
    postBody.persistent = true;
    postBody.priority = 1;
    postBody.scenarioName = mappingData.scenarioName || '';
    postBody.requiredScenarioState = mappingData.requiredScenarioState || '';
    postBody.newScenarioState = mappingData.newScenarioState || '';
}
````

2. When loading stubs from a file, the script checks if the file contains an array of mappings. If it does, it adds each mapping separately. If it doesn't, it adds the single mapping directly.
```javascript
if (Array.isArray(stub.mappings)) {
    stub.mappings.forEach(mapping => { addMapping(mapping); });
} else {
    if (stub.response.bodyFileName) {
        inlineBodyFile(stub);
    } else {
        addMapping(stub);
    }
}
```

Example of stateful behavior json:
```json 
{
    "mappings": [
        {
            "scenarioName": "To do list",
            "requiredScenarioState": "Started",
            "request": {
                "method": "GET",
                "url": "/todo/items"
            },
            "response": {
                "status": 200,
                "body": "<items><item>Buy milk</item></items>"
            }
        },
        {
            "scenarioName": "To do list",
            "requiredScenarioState": "Item added",
            "request": {
                "method": "GET",
                "url": "/todo/items"
            },
            "response": {
                "status": 200,
                "body": "<items><item>Buy milk</item><item>Cancel newspaper subscription</item></items>"
            }
        }
    ]
}
```