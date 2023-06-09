# AWS documentation for POC
A POC implementation of a network configuration capable of supporting a small university

As someone competent in cloud infrastructure, I decided to design a functional web server that could fulfill the basic needs of a smaller university. I tasked myself with setting up a web server that could demonstrate to a university that currently employs physical infrastructure how moving to the cloud could save time and costs. The requirements that I used were a web application, a functional cloud computer, and following the AWS well-architected framework, this includes making the web application functional, load balanced, scalable, highly available, cost-optimized, and high performing.

 I decided to start thinking of what I would make my infrastructure look like as soon as possible, as a result, I came up with a relatively inexpensive setup that would meet the criteria. But not all things are as simple as that. Later on, I made multiple refinements to the layout of my infrastructure as part of the design process.
 

The cost of the infrastructure I would implement can be visualized with the AWS pricing calculator
![unnamed](https://github.com/Cole250/documentation/assets/133917569/c23f01a8-885d-4aa5-9a6e-ea3d9e446898)

The infrastructure implemented were two t.2 micro instances, in the use case of a university, they would have peak hours, and be used for about a year before analysis is run for if they would stick to a hybrid or full cloud model.


Step 1 Basic setup

In any scenario, the function of the VPC (amazon virtual private cloud) is to enable you to start up services inside a private network. This network has similarities to traditional networks, but everyone gets the added benefit of using AWS infrastructure, which has more security and scalability.
It all starts with a blank canvas
:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/bd0b026a-3804-4acf-a2ba-cc87b2ec7459)

This is a representation of the cloud, inside it can fit any service AWS has to offer. But for now, I am going to build the network, the first step I took is creating a VPC.
-	 In the service navigation area of the AWS console,. After typing VPC in the search bar, and clicking on the first option. I now can click on the option “Create a VPC”. Now in the setup process. I need to make sure the “VPC and more” option is selected before continuing, the auto-generate option too needs to be selected right underneath it, but it’s important to choose something memorable. Next, the ipv4 CIDR block, the easiest set to work with right now is 10.0.0.0/16.
This is my Cloud right now:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/6392d59c-196b-4fc1-b4d6-8c77e32b2e82)
-	Choose the number of availability zones as 1, the number of public subnets as 1, and the number of private subnets as 1 for now, close to beneath the choice for the number of private subnets is a drop-down to “customize subnets CIDR blocks”, after opening it change Public subnet CIDR block in us-east-1a to 10.0.0.0/24, and change Private subnet CIDR block in us-east-1a to 10.0.1.0/24. Last, choose NAT gateways in 1 AZ, set the VPC endpoint to none, and keep both DNS hostnames and DNS resolution enabled. After verifying the settings in the preview panel on the right, you can press Create VPC.
This was my current configuration:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/45ffa094-8dc4-4822-8bb6-9744aa09e0e9)
For practice or a refresher, two additional subnets will be created now instead of during the VPC setup process, one private and public. The function of a subnet is to provide a logical divider between resources, with its own range of IP addresses and security. 
-	While in the VPC console, in the left navigation pane, the subnets hyperlink should be right under “Your VPCs”, after clicking on subnets and then creating a subnet, choose the VPC you just created as the VPC ID. For the second public subnet choose the subnet name (something like “project-subnet-public2”), then choose an availability zone different from your other public subnet, to follow the trend of the CIDR blocks, make it 10.0.2.0/24, verify you have done this correctly and click create a subnet. The private subnet follows a similar method, VPC ID: project-vpc, subnet name: “project-subnet-private2”, ipv4 CIDR block: 10.0.2.0/24.
To make the internet accessible to the private subnet, we must manipulate a route table (route tables contain routing rules that control where traffic is going), and each subnet right now has its own route table, we will be changing the connections in the route tables to include both subnets for the private route table and public route table.
-	back in the VPC console, click on route tables in the left navigation pane, select your public route table (mine would be project-rtb-public), then in the lower pane that pops up, choose the routes tab. Something to notice here is that there is a destination route already there with an address that looks like 0.0.0.0/0, it’s target will look something like igw-xxxxxxxxxxxx which is an internet gateway, this is where traffic is being sent. One tab in the navigation pane is “subnet associations”, click on it and below click on “edit subnet associations”, select public subnet 2, and save the association. Select the private route table and associate both private subnets.
This is now your current configuration:
![image](https://github.com/Cole250/documentation/assets/133917569/bbdf623a-4f73-4da9-80f3-62e180e6302a)
- The last step in the network configuration process is to make a quick web security group that will act like a firewall to enable web requests. The first step is to navigate to “security groups” in the left pane, then choose the orange “create security group”. The name I chose will be “Web security group”, the description is “Enable HTTP access”. It’s important here to make sure that the correct VPC is selected again, next, I add an inbound rule, the type is HTTP, the source is “anywhere-IPv4”, and the description is “permit web requests”


Step 2 Web Server Ready

Now that I have my network frame, I can add services, there are many to choose from, but they stray from my criteria. I will be setting up an EC2 (Elastic compute cloud) instance. Like the function of a normal web server, an EC2 instance is a virtual server, which consists of the average server components, a processor, RAM, storage, and other customizable features. The best part about operating a virtual server is that it has multiple payment options, which includes its staple in price optimization, the pay-as-you-go feature, this choice only charges you for what you use, not the infrastructure itself.
-	To set up an instance, I navigate away from the VPC console by typing EC2 in the search bar and clicking on EC2. The orange button labeled “launch instance” is what I click next to enter the setup process for an EC2 instance.
-	for my use case I set the name as “university web server”, next is the machine instance, or AMI (the operating system my server will start up with), which for my use case will be the Ubuntu image, it’s not necessary to change anything more than the machine image in the “Application and OS Images (Amazon Machine Image)” segment. The instance type is t2.micro in the free tier (the free tier offers a year of limited service capabilities for free). The next step can mess with me sometimes, the easiest choice for the key pair is vockey, an aws managed key pair, occasionally, if your EC2 instances are in a different availability zone than Virginia, vockey is not an option, so I had to choose the option to the right “create new key pair”, and the default settings of the RSA key pair type and “.pem” private key file format will be fine. After I click the orange “create key pair” it’s onto the network section.
-	Next to network settings, choose edit, then I change the VPC to the one that was created in the beginning. For the subnet, I use the second public subnet I created, which will remove the possibility that there is something wrong with the private subnets so I can troubleshoot what could go wrong with my instance. Auto-assign public ip stays on, and I now choose the security group I created at the end of step 1 basic setup. After scrolling to “advanced details” and expanding it, I then scrolled to the user data box, and put in basic user data to set up the server, it looks like this:


- #!/bin/bash -xe
- apt update -y
- apt install nodejs unzip wget npm mysql-server -y
- #wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-DEV/code.zip -P /home/ubuntu
- wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCAP1-1-79581/1-lab-capstone-project-1/code.zip -P 
- /home/ubuntucd /home/ubuntu
- unzip code.zip -x "resources/codebase_partner/node_modules/*"
- cd resources/codebase_partner
- npm install aws aws-sdk
- mysql -u root -e "CREATE USER 'nodeapp' IDENTIFIED WITH mysql_native_password BY 'student12'";
- mysql -u root -e "GRANT all privileges on *.* to 'nodeapp'@'%';"
- mysql -u root -e "CREATE DATABASE STUDENTS;"
- mysql -u root -e "USE STUDENTS; CREATE TABLE students(
- id INT NOT NULL AUTO_INCREMENT,
- name VARCHAR(255) NOT NULL,
- address VARCHAR(255) NOT NULL,
- city VARCHAR(255) NOT NULL,
- state VARCHAR(255) NOT NULL,
- email VARCHAR(255) NOT NULL,
- phone VARCHAR(100) NOT NULL,
- PRIMARY KEY ( id ));"
- sed -i 's/.*bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
- systemctl enable mysql
- service mysql restart
- export APP_DB_HOST=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
- export APP_DB_USER=nodeapp
- export APP_DB_PASSWORD=student12
- export APP_DB_NAME=STUDENTS
- export APP_PORT=80
- npm start &
- echo '#!/bin/bash -xe
- cd /home/ubuntu/resources/codebase_partner
- export APP_PORT=80
- npm start' > /etc/rc.local
- chmod +x /etc/rc.local

-	 (This user data is reused and not anything that should be used officially, its main purpose is to show that the web application is not smoke and mirrors)
-	The configurations are reviewed, and I click the orange “launch instance button”. After a few minutes, both checks are passed and my application is accessible, my configuration now looks like this:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/7ba58752-1d49-4fbe-aa06-52b5426715ba)

Step 3 Load Balancing Ready

In this part, I start the process of making the web server highly available, which includes a load balancing service (when there are multiple EC2 instances it moves web requests between them), and an auto-scaling service (starts up new EC2 instances when there is too much load on the current instances). These will allow people trying to view my website to not notice a difference even if it is under high load. 
-	The first thing I can do to begin setting up for an auto-scaling group Is create an AMI (Amazon Machine Image), A machine image is a snapshot of an EC2 instance you have already created, making it easier to start up more instances faster or allowing an auto-scaling group to start up instances how you want. While I am still in the “instances” part of the EC2 console, I select the instance I created and click “actions”, close to the bottom of the drop-down is another dropdown labeled “Image and templates” and create image is what I finally select. I make the image name “ServerAMI” and the description “AMI for university web server”, then I finally press the orange button “create image”.

While it would be easy to make multiple EC2 instances and call it good, it would not make the application highly available, because people would have to navigate to a different URL to access the same thing. However, the application load balancer uses one URL and communicates between multiple instances.
-	To start configuring the load balancer, I navigated to the load balancing section of the left navigation pane and clicked on target groups. Next in "basic configuration", I choose instances, name it “Projectgroup”, select the project VPC, and click on the orange “next” button. The last thing to do is double-check the configuration and click the orange “create target group”. Looking back at the left navigation I click on load balancers, afterwards, I click on “create load balancer” and choose the application load balancer. I make the name “project ELB”, select the VPC I created, two availability zones (the ones that hold both public subnets), and both public subnets in each availability zone. I choose the “web security group” that I created in the first step, next, in listeners and routing I use the HTTP protocol with port 80 and have it forward to the “projectgroup” that I just made.

The last part of making the web server highly available is to create a launch configuration and an auto-scaling group. Launch configurations give auto-scaling groups more instructions on what to do when it needs to scale, like the AMI, the instance type, key pair, etc. However, recently AWS has begun to migrate from launch configurations to “launch templates” which are very similar but have a few extra options.

-	In the auto-scaling groups' option in the left navigation pane of the EC2 console, Near the orange “create auto-scaling group” Is a white button “Launch configurations”. After clicking on it I am taken to the launch configurations page and click the orange “create launch configuration”. I name it “project configuration”, choose the AMI from the beginning of this step, choose a t.2 micro instance, then in the additional configuration in the small “monitoring” section, I “enable EC2 instance detailed monitoring with cloud watch”, select web security group under security groups, and lastly choose vockey or the key pair I already created with my first EC2 instance. I finish creating the launch configuration with the orange button,  still in the launch configurations page, I select the launch configuration I just made and click “actions” and then “create auto-scaling group”.
-	Now in the auto-scaling group configuration, I do the following. I name it “Project auto-scaling group” and click the orange “next”, I select the VPC I made, and both private subnets in “availability zones and subnets”, then I click the “next” button, next is the load balancer. First I “attach to an existing load balancer” and “choose from load balancer target groups”, this is where the target group I just created comes in (I select “projectgroup”). After scrolling down to the bottom and selecting “enable group metrics collection within Cloud watch” I click the orange “next”, I am now in the “configure group size and scaling policies” (step 4), and I chose desired capacity: 1, minimum capacity: 1, maximum capacity: 2, capacity in this reference is how many instances will be deployed. In scaling policies I choose “target tracking scaling policy so I can configure one, I name it “projectscalingpolicy”, the metric is average CPU utilization, and the target value is 70, this will instruct the auto-scaling group to add another instance when the first web server is under high load and remove instances when nothing has been going on for long enough. Finally, I press the orange next button and press the button again, because notifications would be for a real application of this documentation, and they would keep me aware of any scaling that goes on. In step six I click add tag, I make the key “name” and the value “university instance, after clicking the orange “next button which takes me to review, I scroll to the bottom and click “create auto-scaling group”
This is my current configuration:
![unnamed](https://github.com/Cole250/documentation/assets/133917569/a0259ff3-f5bf-4328-9ed7-3cc3f8b65d65)
A SIDE NOTE:
Once I have my auto-scaling group operational, I terminate the instance I made at the beginning, by going to the Instances section in the EC2 console, selecting the instance, clicking actions, and then “terminate instance”, it only takes a few minutes for this to finish.


Refinements And Test Runs

Load testing is the most important step to me, would an inventor not test their own creation?
there are quite a few ways to load test an auto-scaling web server, and AWS has the right tools to do so making it easier to not cross-platform.
- The first thing I do is navigate to cloud9 in the search bar, now in the cloud9 console, I choose the orange “create environment” and enter into configuration. I set the name as “loadtest”, the environment type as a “new EC2 instance”, and the instance will be a t.2 micro, in network settings I choose the connection as “secure shell (SSH), and click the orange “create” button. After the environment is created I select it and choose “open in cloud9”, which will navigate me to the console, I need to install a load testing application to put stress on my instances, to install I type 
“npm install -g loadtest”
 once the command has finished, I run
 “loadtest --rps 2000 -c 1000 -k http://<LoadBalancerDNS>”	(the DNS of the application load balancer goes at the end)
I also have the option to increase the rps (web requests per second) and c (connections) like this 
 “loadtest --rps 8000 -c 4000 -k http://<LoadBalancerDNS>”
From this point on the console updates frequently, and I need to wait up to 7 minutes before the load balancer updates with enough data to initiate a new instance, in the cloud9 console I press ctrl+c to end the command.

 This is an example of what it looks in the ec2 console after the auto scaling process completes, earlier, apporximately 10 minutes ealier, the only running instances were the cloud9 console and the singular university instance, after I used the cloud9 console to put a load on the application load balancer, two more instances were added as per the max instances the auto scaling group is allowed to initialize.
![Screenshot 2023-06-12 134316](https://github.com/Cole250/documentation/assets/133917569/8dab9a18-15b1-4db0-8594-cfe5b1f22f4b)

 This is the auto scaling group activity that initialized all three instances.
 ![Screenshot 2023-06-12 134520](https://github.com/Cole250/documentation/assets/133917569/f0d039bb-e6e8-4236-89c1-0383277d6bb5)

 Lastly, this is the activity present monitored through the application load balancer
 ![Screenshot 2023-06-12 134714](https://github.com/Cole250/documentation/assets/133917569/fb6fb76e-1810-4f7c-8b62-f74501ee677c)

 
By load testing my infrastructure, I found out that my dynamic scaling policy (what I was supposed to set earlier to tell the load balancer when it needs more instances) had stopped existing, it’s important to find flaws like this instead of them happening at a bad time, load testing my architecture was very helpful in this regard.
