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





### X. Make a duplicate of the dashboard for safe-keeping.



### End Enabling Cloudwatch Dashboard to show multi-region metrics


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

### 3.3 Global Accelerator <--this should probably be its own lab4 maybe. 




# Checkpoint 

Proceed to [Lab 4](../lab-4-globalacc)!
