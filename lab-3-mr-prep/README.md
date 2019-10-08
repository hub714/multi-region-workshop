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

### 3.2 DB replication

### 3.3 Global Accelerator <--this should probably be its own lab4 maybe. 

# Checkpoint 

Proceed to [Lab 4](../lab-4-globalacc)!
