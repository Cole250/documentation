# aws documentation for POC
A POC implementation of a network configuration capable of supporting a small university

As someone who is competent around cloud infrastructure, I decided to design a functional web server that could fulfill the basic needs of a smaller university. I tasked myself with setting up a web server that could demonstrate to a university that currently employs physical infrastructure how moving to the cloud could save time and costs. The requirements that I used were a web application, a functional cloud computer, and following the AWS well-architected framework, this includes making the web application functional, load balanced, scalable, highly available, cost optimized, and high performing.

 I decided to start thinking of what I would make my infrastructure look like as soon as possible, as a result I came up with a relatively inexpensive setup that would meet the criteria. But not all things are as simple as that. Later on I made multiple refinements to the layout of my infrastructure as part of the design process.
 
Step 1 basic setup

In any scenario, the function of the VPC (amazon virtual private cloud) is to enable you to start up services inside a private network. This network has similarities to traditional networks, but you get the added benefit of using AWS infrastructure, which has more security and scalability.
We start with a blank canvas:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/bd0b026a-3804-4acf-a2ba-cc87b2ec7459)

This is a representation of the cloud, inside it can fit any service AWS has to offer. But for now we are going to build the network, the first step we take is creating a VPC.
-	 In the service navigation area of the aws console we type VPC in the search bar clicking on the first option. We can now click on the option “create a VPC”, we now move into the setup process. Make sure the “VPC and more” option is selected before continuing, the auto generate option too needs to be selected right underneath it, but you should change the name underneath it from project to a name of your choice. Next, the ipv4 CIDR block, the easiest set to work with right now is 10.0.0.0/16, this is your network right now:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/6392d59c-196b-4fc1-b4d6-8c77e32b2e82)
-	Choose the number of availability zones as 1, the number of public subnets as 1, and the number of private subnets as 1 for now, close to beneath the choice for the number of private subnets is a drop down to “customize subnets CIDR blocks”, after opening it change Public subnet CIDR block in us-east-1a to 10.0.0.0/24, and change Private subnet CIDR block in us-east-1a to 10.0.1.0/24. Last, choose NAT gateways in 1 AZ, set VPC endpoint to none, and keep both DNS hostnames and DNS resolution enabled. After verifying the settings in the preview panel on the right, you can press Create VPC.
This was my current configuration:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/45ffa094-8dc4-4822-8bb6-9744aa09e0e9)
For practice or a refresher, two additional subnets will be created now instead of during the VPC setup process, one private and public. The function of a subnet is to provide a logical divider between resources, with its own range of IP addresses and security. 
-	While in the VPC console, in the left navigation pane, the subnets hyperlink should be right under 
“Your VPCs”, after clicking on subnets and then create subnet, choose the VPC you just created as the VPC ID. For the second public subnet choose the subnet name (something like “project-subnet-public2”), then choose an availability zone different from your other public subnet, to follow the trend of the CIDR blocks, make it 10.0.2.0/24, verify you have done this correctly and click create subnet. The private subnet follows a similar method, VPC ID: project-vpc, subnet name: “project-subnet-private2”, ipv4 CIDR block: 10.0.2.0/24.
To make the internet accessible to the private subnet, we must manipulate a route table (route tables contain routing rules that control where traffic is going), and each subnet right now has its own route table, we will be changing the connections in the route tables to include both subnets for the private route table and public route table.
-	back in the VPC console, click on route tables in the left navigation pane, select your public route table (mine would be project-rtb-public), then in the lower pane that pops up, choose the routes tab. Something to notice here is that there is a destination route already there with an address that looks like 0.0.0.0/0, it’s target will look something like igw-xxxxxxxxxxxx which is an internet gateway, this is where traffic is being sent. One tab in the navigation pane is “subnet associations”, click on it and below click on “edit subnet associations”, select public subnet 2 and save the association. Select the private route table and associate both private subnets.
This is now your current configuration:
![image](https://github.com/Cole250/documentation/assets/133917569/bbdf623a-4f73-4da9-80f3-62e180e6302a)
