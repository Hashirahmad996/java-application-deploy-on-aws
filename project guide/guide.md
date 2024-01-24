> **Project guide**
>
> **Pre-Deployment**

Customize the application dependencies mentioned below on AWS EC2
instance and create the Golden AMI.

1.  AWS CLI

2.  Install Apache Web Server

3.  Install Git

4.  Cloudwatch Agent

5.  Push custom memory metrics to Cloudwatch.

6.  AWS SSM Agent

Numbers 1 & 6 are installed by default on AmazonLinux 2, so install the
rest.

-   Launch an instance in a default VPC and run the following commands

sudo su

yum install -y httpd git

yum install amazon-cloudwatch-agent -y

-   Run this Cloudwatch config wizard and select the defaults, but
    > Ensure to select the memory option when prompted and the cwagent
    > user

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

-   Start the cloudwatch agent

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a
fetch-

config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
-s

-   Verify the cloudWatch agent is running

systemctl status amazon-cloudwatch-agent.service

-   To Push custom memory metrics to Cloudwatch, attach an IAM role to
    > The instance with this AWS-managed policy named
    > CloudWatchFullAccess

-   If you need to test the session manager works also attach
    AmazonSSMFullAccess

AWS managed policy to the existing IAM role

Once all dependencies are installed, create the AMI as shown below

![Graphical user interface, text, application Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image1.jpeg){width="6.54548009623797in"
height="1.4391666666666667in"}

> **VPC Deployment**

1.  Build VPC network ( 192.168.0.0/16 ) for Bastion Host deployment as
    > per the architecture shown above.

Implemented my whole assignment in Canada (central) region. Create
Bastion VPC with this basic configuration

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image2.jpeg){width="6.507417979002625in"
height="2.55in"}

2.  Build VPC network ( 172.32.0.0/16 ) for deploying Highly Available
    > and Auto Scalable application servers as per the architecture
    > shown above.

Create App VPC with this basic configuration

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image3.jpeg){width="6.488459098862642in"
height="2.6549989063867017in"}

3.  Create NAT Gateway in Public Subnet and update Private Subnet
    > associated Route Table accordingly to route the default traffic to
    > NAT for outbound internet connection.

Create all 4 subnets shown for APP VPC. Note route table IDs, AZs and
CIDRs

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image4.jpeg){width="6.497876202974628in"
height="2.125in"}

Create Nat gateway. Note connectivity type, EIP and public subnet

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image5.jpeg){width="6.487962598425197in"
height="2.6949989063867017in"}

See private and public route table rules.

Note 3 route with targets for local, natgw and transit gw in private
route table

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image6.jpeg){width="6.48912510936133in"
height="2.735in"}

Private route table Subnet associations

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image7.jpeg){width="6.49930227471566in"
height="2.59in"}

Public route table in APP VPC with 0.0.0.0/0 route

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image8.jpeg){width="6.494306649168854in"
height="2.535in"}

Public route table Subnet associations

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image9.jpeg){width="6.493159448818898in"
height="2.505in"}

4.  Create Transit Gateway and associate both VPCs to the Transit
    > Gateway for private communication.

Click create transit gateway and add only a name

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image10.jpeg){width="6.491324365704287in"
height="2.91in"}

You need to create 2 transit gw attachment to each VPC. Add name, select
existing transit gw ID, select Bastion & App VPC

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image11.jpeg){width="6.489470691163604in"
height="2.825in"}

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image12.jpeg){width="6.499340551181103in"
height="2.7399989063867016in"}

5.  Create Internet Gateway for each VPC and Public Subnet associated
    > Route Table accordingly to route the default traffic to IGW for
    > inbound/outbound internet connection.

Create 2 Internet GWz and attach to each VPC. Routes are shown with
0.0.0.0/0 in previous step snapshot

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image13.png){width="6.560247156605424in"
height="8.378333333333334in"}

This transit gateway route table will be created by default with these
associations to each VPC

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image14.jpeg){width="6.4883497375328085in"
height="2.63in"}

Afterwards update the private route table in ApP vpc and public route
table in Bastion VPC, with the routes shown in previous snapshot

6.  Create Cloudwatch Log Group with two Log Streams to store the VPC
    > Flow Logs of both VPCs.

To create 2 logroups, Click logroup in cloudwatch and click create log
group. These 2 for Bastion and App VPCs were created with a retention 3
days. No logs streams will be seen until next step

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image15.jpeg){width="6.4938353018372705in"
height="2.78in"}

7.  Enable Flow Logs for both VPCs and push the Flow Logs to Cloudwatch
    > Log Groups and store the logs in the respective Log Stream for
    > each VPC.

To enable flow logs, click Actions and Create flow logs. Repeat for each
VPC

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image16.jpeg){width="6.496060804899388in"
height="2.52in"}

This is bastion VPC flow log settings after creation. Note the
destination Name from previous step

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image17.jpeg){width="6.499268372703412in"
height="2.47in"}

Similar config for APP vpc flow log

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image18.jpeg){width="6.498330052493438in"
height="2.65in"}

These log streams will auto stream once steps above are completed. APP
vpc sample flow logs are Shown below

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image19.jpeg){width="6.492625765529309in"
height="2.935in"}

8.  Create Security Group for bastion host allowing port 22 from public.

Open port 22 and icmp (optional for ping tests)

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image20.jpeg){width="6.4969969378827646in"
height="2.705in"}

9.  Deploy Bastion Host EC2 instance in the Public Subnet with EIP
    associated.

Create Bastion subnet and public route table. Note the CIDR and linked
route table

![A screenshot of a computer Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image21.jpeg){width="6.497040682414698in"
height="3.05in"}

Public route table of the bastion subnet is shown here. The red route is
for a route via the transit gateway to the App VPC and the blue route is
for a route for Internet access in and out of the bastion subnet. The
tgw ID will only appear after setting up the transit gateway

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image22.jpeg){width="6.491719160104987in"
height="2.94in"}

10. Create S3 Bucket to store application specific configuration.

This bucket was created in Canada region with default settings

![Graphical user interface, text, application Description automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image23.jpeg){width="6.497829177602799in"
height="2.495in"}

11. Create Launch Configuration with below configuration.

    1.  Golden AMI

    2.  Instance Type -- t2.micro

    3.  Userdata to pull the code from Bitbucket Repository to document
        > root folder of webserver and start the httpd service.

    4.  IAM Role granting access to Session Manager and to S3 bucket
        > created in the previous step to pull the configuration. (Do
        > not grant S3 Full Access)

    5.  Security Group allowing port 22 from Bastion Host and Port 80
        > from Public.

    6.  Key Pair

Specification shown here. Create a launch template

Referenced previously taken golden ami

![Graphical user interface, text, application, email Description
automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image24.jpeg){width="6.486391076115486in"
height="5.064374453193351in"}

Launch template security group,

![Graphical user interface, text, application, email Description
automatically
generated](vertopal_08cc382a976a4da3987706439f88ed12/media/image25.jpeg){width="6.459244313210848in"
height="6.482707786526684in"}

Userdata clones repo and starts apache

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image26.jpeg){width="6.463566272965879in"
height="4.718541119860017in"}

This is the security group rules for launcg template for reference

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image27.jpeg){width="6.499130577427821in"
height="3.1149989063867016in"}

12. Create Auto Scaling Group with Min: 2 Max: 4 with two Private
    > Subnets associated to 1a and 1b zones.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image28.png){width="6.498040244969379in"
height="5.99in"}

13. Create Target Group and associate it with ASG.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image29.png){width="6.495737095363079in"
height="5.715in"}

14. Create Network Load balancer in Public Subnet and add Target Group
    as target.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image30.png){width="6.497504374453193in"
height="5.425in"}

15. Update route53 hosted zone with CNAME record routing the traffic to
    NLB.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image31.jpeg){width="6.492214566929134in"
height="2.78in"}

> Validation

1.  As DevOps Engineer login to Private Instances via Bastion Host.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image32.png){width="6.499107611548556in"
height="3.2896872265966755in"}

2.  Login to AWS Session Manager and access the EC2 shell from console.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image33.png){width="6.454037620297463in"
height="3.939582239720035in"}

3.  Browse web application from public internet browser using domain
    > name and verify that page loaded.

![](vertopal_08cc382a976a4da3987706439f88ed12/media/image34.jpeg){width="6.395476815398076in"
height="4.219374453193351in"}
