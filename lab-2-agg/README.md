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

1\. Open Amazon Cloudwatch

From the AWS Management Console, select Services and then type **Cloudwatch** into the search bar. You can also find it under the Management and Governance section of the services listing. This will bring up the Amazon Cloudwatch service overview page.

2\. Create a new Cloudwatch dashboard and add a widget

From the menu on the left hand side, select **Dashboards** and then create dashboard. This will prompt you for a name - call it ReInvent App Dashboard and then click Create Dashboard.
* Note - Cloudwatch does not allow spaces in a dashboard name, so will fill in spaces with a hyphen!
![image](https://user-images.githubusercontent.com/23423809/68278028-5babd800-0025-11ea-9a96-b4fc213acdd8.png)

You will be prompted to select a widget to add to your dashboard. Start off by selecting a line widget and click Configure. We will use this widget to map out our number of requests per minute passing through our Application Load Balancer.

![image](https://user-images.githubusercontent.com/23423809/68278128-931a8480-0025-11ea-8d88-721856aeb3dc.png)

We will add our first metric - ALB Requests Per Minute - to the Cloudwatch dashboard.
This is the *RequestCount* metric under the *ApplicationELB* Namespace. Select it by selectin **ApplicationELB -> Per AppELB, per AZ, per TG Metrics** and then selecting the tickbox next to **RequestCount**.

![image](https://user-images.githubusercontent.com/23423809/68278987-594a7d80-0027-11ea-8a43-acd4f8c073d2.png)

Next, select the tab marked *Graphed Metrics*, change the Period to 1 Minute and change the Statistic to *Sum*. (Tip - we know that Sum is the most useful statistic to use, as the [Cloudwatch documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-cloudwatch-metrics.html#load-balancer-metrics-alb) gives us a recommendation like this for each metric).

Give your widget a name by pressing the pencil next to *Untitled Graph*, inputting a name (e.g. ALB Requests Per Minute), click on the little tick-box and click **Create widget**.

![image](https://user-images.githubusercontent.com/23423809/68279497-65830a80-0028-11ea-8c7a-f76970713829.png)

You have now created your first dashboard and added your first widget! Make sure to **Save your dashboard!!** by clicking the button *Save Dashboard*.

3\. Repeat for next set of metrics...


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
