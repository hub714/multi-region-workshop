# Mythical Mysfits: Multi-Region-Workshop

## Lab 3 - Prepping to make it multi region
**steve has slides on this - things u may have forgotten** - it's in chat

In this lab, you will launch your app in another region and then use Global Accelerator to route traffic. 

Here's a reference architecture for what you'll be building:

[TODO] CREATE REF ARCHITECTURE PICTURE
![CodeBuild Create](images/arch-codebuild.png)

Here's what you'll be doing:

[TODO] CREATE TOC
* [Create AWS CodeBuild Project](#create-aws-codebuild-project)
* [Create BuildSpec File](#create-buildspec-file)
* [Test your AWS CodeBuild Project](#test-your-aws-codebuild-project)

### 2.1 Artifact Replication
- Also need to copy over artifacts for deployment
- ECR cross region replication?
- object replication in S3
- codecommit x-region app
- 

### Replicate The app to a second region

    aws cloudformation deploy --stack-name second-region --template-file core.yml --capabilities CAPABILITY_NAMED_IAM --region us-west-2

### 3.2 Replicate the Database

So now that you have a separate stack, we need to set up DynamoDB so that it automatically replicates any data created using the app in the primary region.

There's an easy way to do this - DynamoDB Global Tables. This feature will ensure we always have a copy of our data in both our primary and failover region by continuously replicating changes using DynamoDB Streams. We'll set this up now.

**Note:** In order to setup Global Tables you will need an empty table. For this lab this is not a big issue but if you are migrating from an system with existing data you will need a solution to backup/restore data or migrate from one your old table to a new table with your regions already setup for Global Tables replication. We'll leave this as an exercise ot the reader.

In your source region (double check this) DynamoDB, select the table. It will be named 'Table-' followed by your chosen stack name.

![Configure DynamoDB with Global Tables](../images/03-ddb-global-tables-screen.png)

Next, choose the Global Tables tab from the top and go ahead and create your Global Table and choose your second region - just accept any messages to enable anything it needs and to create any roles it may need as well.

![Configure DynamoDB with Global Tables](../images/03-ddb-global-tables-config.png)

Now that you have created the Singapore Global Table, you can test to see if it is working by creating a new misfit in the primary app you deployed in the second module. Then, look at the DynamoDB table in your secondary region, and see if you can see the record for the ticket you just created:


### Enabling Cloudwatch Dashboard to show multi-region metrics

Now that you have deployed the stack in the secondary region, lets adjust the Cloudwatch dashboard that you created in the previous lab to include these new resources. This will provide visibility to the Mythical and Like services running across both regions on the same dashboard.

If you are unfamiliar with Amazon Cloudwatch, you may consider creating a duplicate of the current Cloudwatch dashboard for safe keeping. That way you can always revert back to the original if you need to.

<details>
    <summary>Instructions: How do I do this?</summary>
    
* Select the Cloudwatch dashboard you wish to duplicate
* Click **Actions** followed by **Save dashboard as...**
* Enter a name for the new dashboard - **BackupOfMyDashboard**
* Click **Save dashboard**

</details>





### X. Edit the widgets to show metrics from the other region

With Amazon Cloudwatch, we have the ability to stack metrics on top of each other in a widget that contains a graph. This will be useful in our case where we are viewing the same metric type, over two resources. We'll do this in the steps below in addition to adding the metrics from the other region.

Hint - see documentation for [Editing a Graph on a Cloudwatch Dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/edit_graph_dashboard.html)

### X.a Edit the ALB widgets

As we are now adding in metrics from two different regions, we must navigate to the secondary region and load the dashboard from there. This is because when referring to metrics within a dashboard, Cloudwatch can only see resources local to that region.

Modify the ALB Requests Per Minute widget to show the metrics from the ALB in the secondary region:

* Open up the [Cloudwatch Dashboards](https://us-west-2.console.aws.amazon.com/cloudwatch/home?region=us-west-2#dashboards:) page and select the dashboard you created in the previous lab
* Change the region (top right of screen) to your Secondary region
* Change the widget to a stacked area
* Add in the **RequestCount** metric to this widget from the ALB
* Change the metric labels to identify the correct region for that metric

<details>
    <summary>Hint with screenshots:</summary>

* Hover over the widget and select Edit in the top right hand corner
![image](https://user-images.githubusercontent.com/23423809/69213104-03420380-0b18-11ea-8cff-e25b09c70fb5.png)
* Change the graph type from a Line to Stacked Area. Then select the All Metrics tab and add in the **requestcount** metric from the ALB
![image](https://user-images.githubusercontent.com/23423809/69213968-83696880-0b1a-11ea-9d71-18a2c1dbfd62.png)
* Select Graphed Metrics and change the label to match the region
![image](https://user-images.githubusercontent.com/23423809/69214232-2d48f500-0b1b-11ea-83a4-2ae1e7dfeade.png)
* Click **Update widget**
    </details>

Modify the ALB HTTP Responses widget to show the metrics from the ALB in the secondary region:

* Change the widget to a stacked area
* Add in the **HTTP 2XX / 4XX / 5XX Count** metrics from the ALB
* Change the metric labels to identify the correct region for that metric
* Ensure the region you put in the label matches the region in the details
* Click **Update widget**

<details>
<summary>Show screenshot:</summary>
    
![image](https://user-images.githubusercontent.com/23423809/69214680-7188c500-0b1c-11ea-8a81-cdb1d549dfb9.png)

</details>
    
    
### X.b Add widgets for the Like and Mythical Services from Secondary region

Following the same process from Lab 2, add a new widget for each of the Like and Mytical services. Modify the titles to be able to easily identify which region they are populating from. You should end up with something like this:

![image](https://user-images.githubusercontent.com/23423809/69214987-46eb3c00-0b1d-11ea-8317-94c55dad4af2.png)

Feel free to move the widgets around the dashboard to suit your style following the instructions in the [https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/move_resize_graph_dashboard.html](Cloudwatch documentation).
Youc can drag widgets around and move them into position wherever you like. You can also add a text widget to show a title, include links to a knowledgebase wiki or internal tooling. Get creative!

[TODO - Add Andy's KPIs / other metrics from X-Ray]

### X.c Add a widget to show statistics from the DynamoDB Global Table

While we're at it, lets create a new widget showing the the Read Capacity Units and Write Capacity Units for our newly created Global Table. Monitoring the table ensures that we have full visibility of the amount of read and write activity which can be useful in troubleshooting efforts. 

Here's how:

* Create a new stacked area graph on the Cloudwatch Dashboard
* Select **DynamoDB**, **Table Metrics**, **[insert table name]**, **ConsumedReadCapacityUnits** and **ConsumedWriteCapacityUnits**
**ConsumedWriteCapacityUnits** for our DynamoDB Global Table.
* Change the statistic type to **Sum**
* Click **Create Widget**

## Important - Save your Cloudwatch Dashboard! ##

### End Enabling Cloudwatch Dashboard to show multi-region metrics



# Checkpoint 

Proceed to [Lab 4](../lab-4-globalacc)!
