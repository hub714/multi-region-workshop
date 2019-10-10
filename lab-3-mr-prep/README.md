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

When architecting a multi-region app we need to be mindful of the fact that most AWS services are Regional. That is, they store their data and configuration in only one region. For our application to function or even be deployed in a second region all of out artifacts will need to be present there first. This is also good DR practice since in the (albeit unlikly) event of anything happening to our artifacts in the primary region, we will have an entirely separate copy of everything we need to get the app running again.

For our application there are a few key artifacts we need to consider replicating:

- Build artifacts for deployment
- Code
- Containers stored in ECR
- Any objects in S3

Let's handle each of these in turn

#### Replicating ECR images

ECR does not currently support cross region replication so we'll have to replicate our images manually.

#### Replicating Objects in S3

S3 supports [cross region replication](https://docs.aws.amazon.com/AmazonS3/latest/dev/replication.html) so we can simply turn this on for the buckets we need.

### Replicating Code in Codecommit

CodeCommit does not support cross region replication natively so we'll have to replicate this manually. See https://aws.amazon.com/blogs/devops/replicate-aws-codecommit-repository-between-regions-using-aws-fargate/


### 3.2 DB replication

### 3.3 Global Accelerator <--this should probably be its own lab4 maybe. 

# Checkpoint 

Proceed to [Lab 4](../lab-4-globalacc)!
