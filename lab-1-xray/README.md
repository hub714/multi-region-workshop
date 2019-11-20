## Distributed Tracing with AWS X-Ray

Observability helps quantify how we are able to meet our availability requirements. An important aspect of observability especially in a microservices architecture is distributed tracing. This enables the ability to profile a request as it passes through our application architecture which may involve one or more services and potentially interactions with backend data stores. Data captured from traces helps teams understand how the application behaves under various conditions and can be incredibly helpful when issues arise. For example, developers can use the data to identify inefficiencies in code and prioritize their sprints. Operations or SRE teams can use the data to diagnose or triage unusual latencies or failures. Infrastructure engineers can use the data to make adjustments to resident scaling policies or resources supporting particular services.

AWS X-Ray is a distributed tracing service that provides an SDK to instrument your applications, a daemon to aggregate and deliver trace data to the X-Ray service, and a dashboard to view a service map which is a visualization of the trace data. If you would like to read more in depth about X-Ray, check out these links to documentation - [What is X-Ray?](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) and [X-Ray Concepts](https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html)

In this lab, you'll continue where our lead developer left off before she was pulled to work on personalization for the application. No surprises there since the PM just got back from re:Invent, and there were many AI/ML sessions in his schedule.

The Mythical Mysfits application is made up of (2) microservices:

1. The Mysfits service serves the Angular front-end application and hosts an API that returns Mysfit profiles from DynamoDB.
2. The Like service tracks the number of likes for a particular mysfit. When a visitor clicks on the heart icon next to a mysfit in the app, a counter for that mysfit's profile is incremented in DynamoDB.

Our lead developer successfully instrumented the Mysfits service, capturing data for inbound http requests and downstream calls to DynamoDB. If you navigate to the [AWS X-Ray dashboard's service map view](http://console.aws.amazon.com/xray/home#/service-map?timeRange=PT30M), you should see some trace data.

We need your help to do the same for the Like service, so we have a more complete picture. Don't worry if you're not a developer, we'll guide you with the lab instructions and provide hints along the way. If you really get stuck, skip to the final hint where we provide the fully instrumented app code. Good luck!

### Instructions

### 1. Add the X-Ray daemon as a sidecar container in the Like service

<details>
<summary>Learn more: What is the X-Ray daemon?</summary>
The AWS X-Ray daemon is an open source software application that listens for traffic on UDP port 2000. It gathers raw segment data and relays it to the AWS X-Ray API. When deployed as a sidecar container with Fargate, the Task IAM role is what authorizes it to communicate with the X-Ray API. The workshop CloudFormation template you ran earlier already created a role that has the necessary permissions. Also, AWS X-Ray provides a managed Docker container image of the X-Ray daemon that you can run as a sidecar. If you'd like to customize the software or container image, you can find the source code on github and a sample Dockerfile in our documentation to build from.

Further reading:

* [X-Ray daemon github repo](https://github.com/aws/aws-xray-daemon)
* [X-Ray daemon permissions](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html#xray-daemon-permissions)
* [X-Ray sample dockerfile](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon-ecs.html#xray-daemon-ecs-build)
* [Application Tracing on Fargate with AWS X-Ray](https://github.com/aws-samples/aws-xray-fargate)
</details>

#### a. Edit the Like service task definition to include the X-Ray daemon container

<details>
<summary>Learn more: Need a refresher on ECS task definitions?</summary>

A task definition is a JSON template that instructs ECS how to launch your container(s). In it you can specify task and container resource requirements, expose listening ports, run one or more container images, and more. If you're familiar with Docker run arguments, they are similar.

Further reading: [ECS Documentation: Task Definitions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html)
</details>

Navigate to [Task Definitions](https://console.aws.amazon.com/ecs/home#/taskDefinitions) in the ECS dashboard  

Find the Like microservice task definition in the list; the name will start with `Multi-Region-Like-Service-` followed by the CloudFormation stack name you set.  

Select the checkbox next to the task definition, and click **Create new revision**.

Scroll down to "Container Definitions" and click **Add container**.

Complete the following fields:

- **Container name** - enter `xray-daemon`
- **Image** - enter `amazon/aws-xray-daemon`
- **Port mappings** - enter `2000` for container port, and select `udp` for protocol

Your configuration will look similar to this:
![X-Ray sidecar](./images/03-xraySidecar.png) TODO ADD NEW SCREENSHOT

Click **Add**

Click **Create**

#### b. Update the Like service ECS service to reference the new task definition

<details>
<summary>Learn more: Need a refresher on ECS services?</summary>

An ECS service maintains a desired number of running ECS tasks.  This is ideal for long running processes like web servers.

Further reading: [ECS Documentation: Services](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html)
</details>

Navigate to [Clusters](https://console.aws.amazon.com/ecs/home#/clusters) in the ECS dashboard.  

Click on your workshop ECS cluster; the name will start with `Cluster-` followed by the CloudFormation stack name.  

Check the checkbox next to the **Like** service and click **Update**.

Configure the following fields:

- **Task Definition** - select the latest from the dropdown menu for revision
- **Force new deployment** - check this box

Your configuration will look similar to this:
![Update Like service](./images/03-updateLikeService.png)

Leave all other fields as they are and keep clicking **Next step** until you reach the review page, then click **Update Service**

Click **View Service** and you should see the deployment begin. This will take a few minutes, so feel free to move on to the next step where you'll begin to instrument the Like service.

### 2. Instrument the Like service code using the AWS X-Ray SDK and Cloud9

<details>
<summary>Learn more: The AWS X-Ray SDK</summary>

AWS provides X-Ray SDKs for many popular programming languages such as python, javascript, go, java, etc. The SDK provides interceptors to trace incoming HTTP requests, client handlers to instrument AWS SDK clients used to call other AWS services, and an HTTP client to instrument calls to other http web services. You can also patch certain supported libraries such as database clients. Support will vary with each language specific SDK, so consult the documentation to read more. Since Mythical Mysfits services are based on Flask, you'll use the python SDK. Additionally, the X-Ray SDK for Python includes middleware that traces incoming requests for Flask (and Django) frameworks. Convenient! If you aren't using either of these, but still use python for your implementation, you can use the python SDK to manually instrument segments.

Further reading:

* [What is X-Ray?](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)
* [AWS X-Ray SDK for Python](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python.html)

</details>

When instrumenting an app with the X-Ray SDK, you first need to install the SDK. For convenience, that was already done for you during workshop bootstrap. If you look at `requirements.txt` for the Like service, you'll see we include the `aws-xray-sdk` when the container image is built.

You should already have the Cloud9 IDE open from bootsrapping the workshop environment in the last lab. The bootstrap script cloned the Like service code to your Cloud9 IDE. Expand on the Like service folder in the directory tree to the left; the name will start with `like-service-` followed by the CloudFormation stack name and more text.

![Cloud9 Like Service Code](./images/01-02a-likeService.png)

Expand the `service` folder and double-click on `mysfits_like.py` to load it in the editor pane. Your teammate commented the code to help guide you. You'll also notice `TODO` statements following those comments, that's where you'll add lines of code to accomplish the todo. Take a moment to review the codebase and move on to the first step once you're ready.

#### a. Import helper functions and classes from the X-Ray SDK to trace incoming HTTP requests for Flask applications and downstream calls to DynamoDB

Add (3) lines of code to:

1. Load the X-Ray recorder class from the SDK's core module
2. Load the X-Ray patch function from the SDK's core module
3. Load the middleware function for flask apps

Note: There are a couple approaches to patching client libraries using either the `patch_all` or `patch` modules. We choose the latter for specificity, but feel free to use the easy button that is the former.

Here is some documentation to help you figure it out:

* [X-Ray Python Middleware](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-middleware.html)
* [Patching libraries](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-patching.html)
* [General AWS X-Ray SDK for Python API reference](https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/index.html)

<details>
<summary>HINT: Completed imports</summary>

```
# Load functions/classes from aws xray sdk to instrument this service to trace incoming 
# http requests and downstream aws sdk calls. This includes the X-Ray Flask middleware
# [TODO] load x-ray recorder class
# [TODO] load x-ray patch function
# [TODO] load middleware function for flask
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```

</details>

#### b. Configure the X-Ray recorder

<details>
<summary>Learn more: X-Ray recorder configuration</summary>

The X-Ray recorder can be customized by setting class attributes. For example, you can name your service segments, enrich traces data with additional metadata by including service plugins, set the address/port of your daemon process (if not using the default of 127.0.0.1/udp), and more.

Further reading:

* [Configuring the X-Ray SDK for Python](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-configuration.html)
* [AWS X-Ray SDK API Reference - Configure Global Recorder](https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/configurations.html)

</details>

Configure (3) attributes in the xray_recorder class to:

1. Set the name of the service to be 'Like Service'
2. Enable the ECS service plugin for additional metadata to be added to the trace
3. Configure the recorder behavior when instrumented code attempts to record data when no segment is open. The behavior we want is to log an error but continue.

Here is some documentation to help you figure it out:

* [Service Plugins](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-configuration.html#xray-sdk-python-configuration-plugins)
* [Recorder Configuration in Code](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-configuration.html#xray-sdk-python-middleware-configuration-code)
* [AWS X-Ray SDK API Reference - Configure Global Recorder](https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/configurations.html)

<details>
<summary>HINT: Completed xray_recorder configuration</summary>

```
# Configure xray_recorder class to name your service and load the ECS plugin for 
# additional metadata.
# [TODO] configure the x-ray recorder with a service name and load the ecs plugin
plugins = ('ecs_plugin',)
xray_recorder.configure(
  service = 'Like Service',
  plugins = plugins,
  context_missing='LOG_ERROR'
)
```

Note: In case you're wondering why there's a trailing comma after `'ecs_plugin'`, it's because plugins is a tuple, and in Python a single value tuple or singleton requires a comma.
</details>

#### c. Patch AWS SDK clients to enable tracing of downstream calls to DynamoDB

Earlier, you imported the `patch` function from the X-Ray SDK core. Use that to patch boto3 which is used by the mysfitsTableClient.

Here is some documentation to help you figure it out:

* [Patching Libraries](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-patching.html)

<details>
<summary>HINT: Completed boto3 patching</summary>

```
# Configure X-Ray to trace service client calls to downstream AWS services
# [TODO] patch the boto3 library
libraries = ('boto3',)
patch(libraries)
```

Note: In case you're wondering why there's a trailing comma after `'boto3'`, it's because libraries is a tuple, and in Python a single value tuple or singleton requires a comma.

</details>

#### d. And finally, configure the Flask middleware

Instantiate the Flask middleware to enable tracing.

Here's is a link to documentation to help you figure it out:

* [Adding Flask Middleware](https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-middleware.html#xray-sdk-python-adding-middleware-flask)

<details>
<summary>HINT: Completed enabling Flask middleware</summary>

```
# Instantiate the Flask middleware
# [TODO] configure middleware with the flask app and x-ray recorder
XRayMiddleware(app, xray_recorder)
```

Note: In case you're wondering why there's a trailing comma after `'boto3'`, it's because libraries is a tuple, and in Python a single value tuple or singleton requires a comma.

</details>

#### e. Checkpoint

You made it! The Like service should be instrumented. Reveal the final hint to compare your work.

<details>
<summary>FINAL HINT: SPOILERS AHEAD - Fully instrumented Like service code</summary>

```
#!/usr/bin/python
from __future__ import print_function
import os
import sys
import logging
import random
from urlparse import urlparse
from flask import Flask, jsonify, json, Response, request, abort
from flask_cors import CORS
import mysfitsTableClient

# Load functions/classes from aws xray sdk to instrument this service to trace incoming 
# http requests and downstream aws sdk calls. This includes the X-Ray Flask middleware
# [TODO] load x-ray recorder class
# [TODO] load x-ray patch function
# [TODO] load middleware function for flask
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

if 'LOGLEVEL' in os.environ:
    loglevel = os.environ['LOGLEVEL'].upper()
else:
    loglevel = 'ERROR'

logging.basicConfig(level=loglevel)

# Configure xray_recorder class to name your service and load the ECS plugin for 
# additional metadata.
# [TODO] configure the x-ray recorder with a service name and load the ecs plugin
plugins = ('ecs_plugin',)
xray_recorder.configure(
  service = 'Like Service',
  plugins = plugins,
  context_missing='LOG_ERROR'
)

# Configure X-Ray to trace service client calls to downstream AWS services
# [TODO] patch the boto3 library
libraries = ('boto3',)
patch(libraries)

app = Flask(__name__)
CORS(app)
app.logger

# Instantiate the Flask middleware
# [TODO] configure middleware with the flask app and x-ray recorder
XRayMiddleware(app, xray_recorder)

# The service basepath has a short response just to ensure that healthchecks
# sent to the service root will receive a healthy response.
@app.route("/")
def health_check_response():
    return jsonify({"message" : "This is for health checking purposes."})

@app.route("/mysfits/<mysfit_id>/like", methods=['POST'])
def like_mysfit(mysfit_id):
    app.logger.info('Like received.')
    if os.environ['CHAOSMODE'] == "on":
        n = random.randint(1,100)
        if n < 30:
            app.logger.warn('WARN: simulated 500 activated')
            abort(500)
        elif n < 60:
            app.logger.warn('WARN: simulated 404 activated')
            abort(404)
        app.logger.warn('WARN: This thing should NOT be left on..')
    
    service_response = mysfitsTableClient.likeMysfit(mysfit_id)
    flask_response = Response(service_response)
    flask_response.headers["Content-Type"] = "application/json"
    return flask_response

# Run the service on the local server it has been deployed to
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

</details>

Now you are ready to check in your code and let the CI/CD pipeline build revised container images and re-deploy the Like service with Fargate.

### 3. Deploy the changes you made to the Like service

Since Mythical Mysfits moved to a microservices architecture, it was apparent that an automated CI/CD pipeline was necessary in order to remain agile. The dev team adopted [AWS CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html) which coordinates a few tasks:

1. Watches for changes in the source repository which is [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)
2. Leverages [AWS CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html) to build new container images for the revised source code and pushes the created image to ECR.
3. Deploys the new image in ECR by updating the ECS Fargate service.

#### a. Check in Like service code to kick off the pipeline

In your Cloud9 terminal, navigate to the Like service folder.

```
$ cd ~/environment/like-service-[PUSH TAB TO AUTO COMPLETE AND PRESS ENTER]
```

Commit your updated code and push to master. If you're not familiar with git commands, expand the hint below for step by step.

<details>
<summary>HINT: Git commands step by step</summary>

Check that the like service code was modified.
```
$ git status
```
Add file to staging.
```
$ git add service/
```
Commit the file to your local repository.
```
$ git commit -m "instrumented like service with xray"
```
Push the commit to CodeCommit (remote repository)
```
$ git push origin master
```

The commands and output should look similar to this:
![Push committed code](./images/01-03-commitInstrumented.png)

</details>

The pipeline will take a few minutes to complete, so feel free to move on to the next step. If you want to watch the pipeline, navigate to the [CodePipeline dashboard](https://console.aws.amazon.com/codesuite/codepipeline/pipelines) and click on the pipeline for the Like service. When it's completed, it will look similar to the screenshot below.

![CodePipeline for Like service](./images/01-03-likeServicePipeline.png)

### 4. Test your configuration

Now that you've instrumented the like service, you should see additional trace data being reported to the service map in the X-Ray console whenever users use the like functionality in the application. This will include inbound http requests to the X-Ray service as well as downstream calls to DynamoDB when it increments the like counters for each mysfit liked.

#### a. Confirm you have a complete service map for the Mythical Mysfits application

Navigate to the [AWS X-Ray dashboard service map view](http://console.aws.amazon.com/xray/home#/service-map?timeRange=PT30M)

Open a new browser tab and load the Mythical Mysfits application by visiting the ALB's DNS name. The load balancer's DNS name is one of the outputs from the CloudFormation template you ran as a part of workshop setup. The bootstrap script you ran writes these outputs to a local JSON file in your Cloud9 IDE. Run the following command in your Cloud9 terminal to get the load balancer's DNS name:

```
$ cat ~/environment/multi-region-workshop/cfn-output.json | grep LoadBalancerDNS
```

When the page loads, you should see a grid of mysfits and notice a heart icon in the bottom right corner of each box. Click on a few hearts for a few mysfits to generate some traffic to the Like service. The service was launched in chaos mode which randomly returns 404s and 500s, so you'd see more interesting data in the X-Ray service map. Keep clicking on the hearts until it lights up orange for a few.

<details>
<summary>Note: Javascript console</summary>
If you open the Javascript console in your browser (e.g. in Chrome, you can find this in View->Developer->Javascript Console), you will see the requests and potential outcomes being successful or not.
</details>

TODO ADD SCREENSHOT OF LIKED MYSFITS

Once you've liked a few mysfits, return to the tab with the X-Ray service map and you should see a service map representative of the Mythical Mysfits application, something like this -

![Completed Service Map](./images/01-04a-completedServiceMap.png)

Take some time to explore the service map a bit more. Note the information you can glean by clicking on each service. Also, explore the raw trace data by clicking on **Trace** in the left menu.

### 5. Reduce the signal from the noise

One thing you'll notice is an abundance of GET requests to the Like service which doesn't add up since the like funtionality is based on POST requests. These GETs are health checks from the ALB, which skews the statistics. Filter expressions to the rescue, and they do exactly as the name implies. It's an expression that filters based on given criteria, e.g. service name, errors, src/dst relationships, annotations. Feel free to experiment with filter expressions by entering them into the search bar in the X-Ray dashboard; for your reference - [filter expression documentation](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-filters.html)

Filter expressions can also be used to group traces. This is important because by creating a group, X-Ray will output the approximate trace counts for a given filter expression as a CloudWatch metric. Subsequently, you can create CloudWatch alarms or use these numbers in an operational dashboard, as appropriate. For example, you could create a trace group that filters out throttling (i.e. 429 error codes) to understand whether a service is overwhelmed.

#### a. Filter POST requests to the Like Service

Create a trace group using a filter expression that extracts POST requests to the Like service. The generated CloudWatch metric will be an additional data point to help indicate service health.

Note: There is a basic utility in the lab-1-xray/util folder that will generate artificial traffic to the like service as you work on implementing the filters. This may be more convenient than manually clicking through the website to generate requests. Choice is up to you. To use the utility, run this command -

```
$ python ~/environment/multi-region-workshop/lab-1-xray/util/ryder.py
```

`Ctrl-C` will kill the process.

Here is some documentation to help you figure out the filter expressions:

* [Filter expression documentation](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-filters.html)


<details>
<summary>HINT: Detailed step by step</summary>

Click on **Create group** in the dropdown menu next to the X-Ray dashboard's filtering search bar.

![Create group](./images/01-05a_createGroup.png)

Enter a name, e.g. `like-service`

Enter `service("Like Service") AND http.method = "POST"` into the filter expression field

Click **Create**

![Create group answer](./images/01-05a_createGroupAnswer.png)

</details>

#### b. Filter on HTTP error codes

Create a trace group using filter expressions to catch 404s and 500s; the X-Ray service refers to these as errors and faults, respectively. The generated CloudWatch metric will be an additional data point to help indicate service health and potentially be used as an alarm for notifications or even automated failover.

Here is some documentation to help you figure it out:

* [Filter expression documentation](https://docs.aws.amazon.com/xray/latest/devguide/xray-console-filters.html)

<details>
<summary>HINT: Detailed step by step</summary>

Click on **Create group** in the dropdown menu next to the X-Ray dashboard's filtering search bar.

![Create group](./images/01-05a_createGroup.png)

Enter a name, e.g. `like-service-errors-faults`

Enter `service("Like Service") { error = true OR fault = true }` into the filter expression field

Click **Create**

</details>

### Checkpoint
Congratulations!!!  You've successfully instrumented the Like service to enable tracing. You also used filter expressions to group important trace data. This data gets reported to CloudWatch as a metric which you can use for operational dashboards and setting up alarms. On to the next lab!

Proceed to [Lab 2](../lab-2-agg)!

[*^ back to top*](#distributed-tracing-with-AWS-X-Ray)

## Participation

We encourage participation; if you find anything, please submit an [issue](https://github.com/aws-samples/amazon-ecs-mythicalmysfits-workshop-STAGING/issues). However, if you want to help raise the bar, submit a [PR](https://github.com/aws-samples/amazon-ecs-mythicalmysfits-workshop-STAGING/pulls)!

## License

This library is licensed under the Apache 2.0 License.
