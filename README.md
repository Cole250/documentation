# aws documentation for POC
A POC implementation of a network configuration assigned to me by a university

As someone who is competent around cloud infrastructure, I decided to design a functional web server that could fulfill the basic needs of a smaller university. I tasked myself with setting up a web server that could demonstrate to a university that currently employs physical infrastructure how moving to the cloud could save time and costs. The requirements that I used were a web application, a functional cloud computer, and following the AWS well-architected framework, this includes making the web application functional, load balanced, scalable, highly available, cost optimized, and high performing.

 I decided to start thinking of what I would make my infrastructure look like as soon as possible, as a result I came up with a relatively inexpensive setup that would meet the criteria. But not all things are as simple as that. Later on I made multiple refinements to the layout of my infrastructure as part of the design process.

Step 1 basic setup
In any scenario, the function of the VPC (amazon virtual private cloud) is to enable you to start up services inside a private network. This network has similarities to traditional networks, but you get the added benefit of using AWS infrastructure, which has more security and scalability.
We start with a blank canvas
