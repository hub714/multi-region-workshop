# Mythical Mysfits: Multi-Region-Workshop

## Lab 5 - Load test your multi-region application
Now that we have the Global Accelerator set up and targetting our two different Load Balancers residing in each region, lets send some test traffic to it. The aim of this lab is to learn how to use the Global Accelerator to manipulate traffic flows and then use this method to direct traffic for our Mysfits service between regions. This will be useful when there is a need to failover between our regions to meet our business requirements.
One of the benefits of using the Global Accelerator in this scenario is that we do not need to wait for DNS TTL's (Time To Live) to expire, nor rely on them. Instead the Global Accelerator provides a single DNS endpoint with two A-records behind it. We only need to send traffic to the single DNS endpoint for the traffic manipulation to be effective.
In addition, we will learn how to use the Global Accelerator Health Checks to automatically direct traffic away from a region where the application is showing an unhealthy state over to another region where the app is healthy.

To do this, we will use [Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html) (AB) to generate some HTTP requests to our Global Accelerator Endpoint from our Cloud9 environment. Apache Bench is a simple command line based tool that can be used to benchmark a webserver.

At a high-level, during this lab we will -
* Run AB against the stack in region A to test it is working correctly
* Run AB against the stack in region B to test it is working correctly
* Set the Traffic Dials within our Global Accelerator Endpoint group to split traffic 50% to each region
* Manually failover the Traffic Dial between Regions
* Artificially "break" the application in one region to force failover by the Global Accelerator



### 5.1 Run AB against the stack in region A to test it is working correctly
Navigate to the Global Accelerator Listener and edit the Endpoint groups Traffic Dial to send 100% traffic to Region A

![image](https://user-images.githubusercontent.com/23423809/68568370-401f4380-0410-11ea-9fb9-4804916f3520.png)

Enter the following command to send some HTTP requests using Apache Bench to our application, via the Cloud9 CLI

`watch ab http://<Insert your Global Accelerator Endpoint>/`

For example: **watch ab http://a174d65be73386799d.awsglobalaccelerator.com/**

![image](https://user-images.githubusercontent.com/23423809/68568756-52e64800-0411-11ea-8dcd-429d922b1fea.png)

Apache Bench is now sending HTTP requests to our endpoint and will continue to do so until we stop teh watch process. Lets leave this running for a couple of minutes - the aim is to see that our Cloudwatch metrics are populating. *(Note - you can press **Control+C** to stop the test now if you want and resume later)*

Next, navigate to the Cloudwatch Dashboard that you created in Lab 2. You should see the different widgets that you have set up within your dashboard begin to have data points in them for Region A. You may need to change the refresh intervel to auto-refresh every 10s and the timeframe to **custom (30m)** to see the new metrics come in.

![image](https://user-images.githubusercontent.com/23423809/68569556-3c40f080-0413-11ea-8364-c2b9759b5c90.png)

Once you see the widgets come to life within the Cloudwatch dashboard, this step is complete. You have confirmed that the application in Region A is working correctly *and* that your Cloudwatch dashboard is populating with data correctly. Horray! Now navigate back to Cloud9 and stop the test by pressing **Control+C** in the terminal window.

### 5.2 Run AB against the stack in region A to test it is working correctly
Next, we will run the same tests as in the previous step, but for Region B.

Navigate to the Global Accelerator Listener and edit the Endpoint groups Traffic Dial to send 100% traffic to Region B

**SCREENSHOT TO BE UPDATED**
![image](https://user-images.githubusercontent.com/23423809/68568370-401f4380-0410-11ea-9fb9-4804916f3520.png)

Enter the following command to send some HTTP requests using Apache Bench to our application, via the Cloud9 CLI

`watch ab http://<Insert your Global Accelerator Endpoint>/`

For example: **watch ab http://a174d65be73386799d.awsglobalaccelerator.com/**

![image](https://user-images.githubusercontent.com/23423809/68568756-52e64800-0411-11ea-8dcd-429d922b1fea.png)

Apache Bench is now sending HTTP requests to our endpoint and will continue to do so until we stop teh watch process. Lets leave this running for a couple of minutes - the aim is to see that our Cloudwatch metrics are populating. *(Note - you can press **Control+C** to stop the test now if you want and resume later)*

Next, navigate to the Cloudwatch Dashboard that you created in Lab 2. You should see the different widgets that you have set up within your dashboard begin to have data points in them for Region B. At the same time, you'll see traffic drop-off for the Region A widgets.

Once you see the widgets come to life for Region B within the Cloudwatch dashboard, this step is complete. You have confirmed that the application in Region B is working correctly *and* that your Cloudwatch dashboard is populating with data correctly. Double horray! Now navigate back to Cloud9 and stop the test by pressing **Control+C** in the terminal window.

### 5.3 Set the Traffic Dials within our Global Accelerator Endpoint group to split traffic 50% to each region

Navigate back to the Global Accelerator Listener and modify the Endpoint groups to send 50% of traffic to Region A and 50% to Region B.

![image](https://user-images.githubusercontent.com/23423809/68570101-b1f98c00-0414-11ea-8d75-01a1a168a693.png)
