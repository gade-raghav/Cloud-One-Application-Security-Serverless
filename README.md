# Cloud-One-Application-Security-Serverless

## Description

Trend Micro Cloud One provides **Application Security** which helps in detection and protection for modern applications and APIs built on your container, serverless, and other computing platforms. In this documentation, we will be demonstrating how we can implement it on **serverless** applications.


We will be going through the following steps :

- Application Group Creation

- Add Layer to Lambda and Set Credentials

- Creating Policy, setting it to detection mode for all modules

- Enabling Required Modules

- Modifying the Error/Block page

- Monitoring the events 
 
## Pre-requisites

- AWS Account (Free tier account with full access is suggested)

- Trend Micro Cloud One Account ([Free trial account](https://cloudone.trendmicro.com/))

- This documentation assumes that Lambdas are already available

#### Let's Start

### Application Group Creation 

1. log in to Cloud One Account and select Application security. 

![Application Security](/images/applicationsec.png)

2. Create a new group.

![Create Group](/images/groupcreate.png)

3. Note the Group Credentials (Key and Secret) found in settings.

![Credentials](/images/appseccredentials.png)


### Add Layer to Lambda

There are two approaches to add a layer to lambda functions :
 
- Using Custom Runtimes (No further changes to code required) 

1. Add a layer to function

Navigate to Lambda Functions Configuration tab on AWS Console and click on Add a layer.
![Add Layer](/images/layers.png)

Click Specify an ARN and enter it.
You can find the appropriate LayerVersionARN on [this page](https://cloudone.trendmicro.com/docs/application-security/downloads/).
![arn](/images/arn.png)


Navigate to AWS Lambda function and select Edit in Runtime Settings. Select Custom runtime and click save.
![Custom Runtime](/images/customrt.png)

In Environment Variables section add TREND_AP_KEY and TREND_AP_SECRET (Group Credentials).
![Custom Runtime Vars](/images/crvars.png)

Sometimes we may not be able to use Custom Runtime. In that case, we can use AWS Runtimes( eg: Python, Node.js, Ruby, etc).

However, in case we are using AWS Runtime, we are supposed to add the following Environment Variables and make few changes to the code.
![Vars](/images/lambdavars.png)

```
TREND_AP_KEY: < key from Application Security Dashboard >

TREND_AP_SECRET: < secret from Application Security Dashboard >

TREND_AP_READY_TIMEOUT: 30

TREND_AP_TRANSACTION_FINISH_TIMEOUT: 10

TREND_AP_MIN_REPORT_SIZE: 1

TREND_AP_INITIAL_DELAY_MS: 1

TREND_AP_MAX_DELAY_MS: 100

TREND_AP_HTTP_TIMEOUT: 5

TREND_AP_PREFORK_MODE: False

TREND_AP_CACHE_DIR: /tmp/trend_cache

TREND_AP_LOG_FILE: STDERR
```

In function code, add the following lines:
for **python3.6**
```
import trend_app_protect.start
from trend_app_protect.api.aws_lambda import protect_handler

@protect_handler
def handler(event, context):

    # Your application code here

    return {
        'statusCode': 200,
        'body': 'Hello from Lambda',
    }
```

for **nodejs10.x**

```
nodejs10.x
var trend_app_protect = require('trend_app_protect');

var _handler = async (event) => {

    // Your application code here

    return {
        statusCode: 200,
        body: 'Hello from Lambda',
    };
};

// Export wrapped handler
exports.handler = trend_app_protect.api.aws_lambda.protectHandler(_handler);
```


### Creating Policy, setting it to detection mode for all modules

Application Security detects and offers protection against a wide range of attacks at runtime. We can get an analysis of all requests made to the application and decide whether to allow it or to take protective measures. Set it to *Report* for all modules to detect all the malicious requests made. 

![Inital](/images/modules.png)


### Enabling Required Modules 

There are many module policies offered by Application Security and we can enable/disable them depending upon the requirement. In case we do not require Malicious File Upload module, we can turn it off.

![deactivate module](/images/deactivatemodule.png)

![deactivate module2](/images/deactivatemodule2.png)



### Modifying the Error/Block page

Application Security Console gives a Block Page Template by default which can be edited to match requirements. The following image is for reference:

![Block page template](/images/bpt.png)


### Monitoring the events 

After completing the setup, we are all set to monitor the requests made to our application and change the module policies accordingly.

Set the Illegal File Access policy to Report and make illegal file access (For demo purpose).
![reportmodule](/images/detected.png)

It has reported the request, however, it will only block the request when we set it to Mitigate. The following request has been blocked.
![change mitigate](images/changepolicy.png)

![blocked](/images/attackblocked.png)










