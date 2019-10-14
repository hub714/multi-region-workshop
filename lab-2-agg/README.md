# Mythical Mysfits: Multi-Region-Workshop

## Lab 2 - Gather AWS Metrics

In this lab, you will start the process of aggregating metrics to understand the health of your application so you can make informed decisions about when to fail over to a different region. We will use an Amazon Cloudwatch Dashboard, amongst other stuff.

Our Cloudwatch dashboard should include metrics from the key components of our system and application. In this case, the metrics we should display on a dashboard are the following:

* Fargate task capacity (CPU / Mem)
* Number of active users using application
* ALB requests per minute
* ALB average latency per request
* KPI that is TBD as custom metric from application
* X-Ray exposed metric to dashboard?

Here's a reference diagram showing the metrics that we'll be putting onto a dashboard
[TODO] ADD IN PRETTY DIAGRAM HERE

Here's what you'll be doing:

[TODO] CREATE TOC
* Create Amazon Cloudwatch Dashboard
* Add metrics to the dashboard
* Save the dashboard and share with another team (TBD)




### 2.1 Create Amazon Cloudwatch Dashboard
From the AWS Management Console, select Services and then type Cloudwatch into the search bar. You can also find it under the Management and Governance section of the services listing.
Select Dashboards on the left hand side and then create dashboard. This will prompt you for a name - call it ReInvent App Dashboard, followed by Create Dashboard. * Note - Cloudwatch does not allow spaces in a dashboard name, so will fill in spaces with a hyphen!
[insert image here]
You will be prompted to select a widget to add to your dashboard. Start off by selecting a line widget. We will use this widget to map out our number of requests per minute passing through our Application Load Balancer.
[insert image here]
Select the metric in question (add stuff about ALB metric here).

Save the dashboard!

Add the remaining metrics to the dashboard using the following widget types for each metric:
* number of active users within applcation - number widget
* number of active users within applcation - line widget
* add a text box to show the current live region

(Include remaining widgets)
(Insert image of what target dashboard should look like, or similar to)


### 2.2 HEADER

# Checkpoint 

Proceed to [Lab 3](../lab-3-mr)!
