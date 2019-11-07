# Mythical Mysfits: Multi-Region-Workshop

## Lab 4 - Global Accelerator and Manual Failover

In this lab, you will use Global Accelerator to route traffic...[TODO]

Here's a reference architecture for what you'll be building:

[TODO] CREATE REF ARCHITECTURE PICTURE

Here's what you'll be doing:
* Create an Accelerator
* Add Listeners
* Add Endpoint Groups
* Add Endpoints
* Test your Accelerator
* Manual Failover

### 4.1 Create an Accelerator

* Open the [Global Accelerator](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#GlobalAcceleratorDashboard:) console. 
* Choose **Create accelerator**. 
* Provide a name for your accelerator. 
* Choose **Next**.

[TODO] add screenshot

### 4.2 Add Listeners

* **Ports**: Enter 80.
* **Protocol**: Choose TCP.
* Client affinity: Leave as None.
* Choose **Next**.

[TODO] add screenshot

### 4.3 Add Endpoint Groups

* **Region**: Choose the primary region that you deployed the application in.
* **Traffic dial**: Leave as 100.
* Choose **Add endpoint group**.
* **Region**: Choose the seconday region that you deployed the application in.
* **Traffic dial**: Enter 0.
* Choose **Next**.

[TODO] add screenshot

### 4.4 Add Endpoints

* Under the primary region endpoint group, choose **Add endpoint**.
* **Endpoint type**: Choose Application Load Balancer.
* **Endpoint**: Choose the ELB.
* Under the secondary region endpoint group, choose **Add endpoint**.
* **Endpoint type**: Choose Application Load Balancer.
* **Endpoint**: Choose the ELB.
* Choose **Create accelerator**.

[TODO] add screenshot

### 4.5 Test your Accelerator

* Wait for the Status of your Accelerator to go from In progress to **Deployed**.
* Click on the name of your Acceleator. Check the that the Status of the Listener is **All healthy**.
* In the confiuration panel, copy either one of the IP addresses under **Static IP address set**.
* Test the static IP address in your browser. You should see...
* Click your Listener ID.
* Click the radio button next to the Endpoint group ID for your primary region. Choose **Edit**. 
* Change **Traffic dial** to 100. Choose **Save changes**.
* Click the radio button next to the Endpoint group ID for your secondary region. Choose **Edit**. 
* Change **Traffic dial** to 0. Choose **Save changes**.
* If the Traffic dial values don't reflect your changes, try pressing the refresh button and the top right.
* Test the static IP address in your browser. You should see...

[TODO] add screenshot

### 4.6 Manual Failover

[TODO]

# Checkpoint 

[TODO] You have done...

Proceed to [Lab 5](../lab-5-loadtest)!
