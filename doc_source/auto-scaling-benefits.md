# Amazon EC2 Auto Scaling benefits<a name="auto-scaling-benefits"></a>

Adding Amazon EC2 Auto Scaling to your application architecture is one way to maximize the benefits of the AWS Cloud\. When you use Amazon EC2 Auto Scaling, your applications gain the following benefits:
+ Better fault tolerance\. Amazon EC2 Auto Scaling can detect when an instance is unhealthy, terminate it, and launch an instance to replace it\. You can also configure Amazon EC2 Auto Scaling to use multiple Availability Zones\. If one Availability Zone becomes unavailable, Amazon EC2 Auto Scaling can launch instances in another one to compensate\.
+ Better availability\. Amazon EC2 Auto Scaling helps ensure that your application always has the right amount of capacity to handle the current traffic demand\. 
+ Better cost management\. Amazon EC2 Auto Scaling can dynamically increase and decrease capacity as needed\. Because you pay for the EC2 instances you use, you save money by launching instances when they are needed and terminating them when they aren't\.

**Contents**
+ [Example: Cover variable demand](#autoscaling-benefits-example)
+ [Example: Web app architecture](#autoscaling-design-example)
+ [Example: Distribute instances across Availability Zones](#arch-AutoScalingMultiAZ)
  + [Instance distribution](#AutoScalingBehavior.Rebalancing)
  + [Rebalancing activities](#AutoScalingBehavior.InstanceUsage)

## Example: Cover variable demand<a name="autoscaling-benefits-example"></a>

To demonstrate some of the benefits of Amazon EC2 Auto Scaling, consider a basic web application running on AWS\. This application allows employees to search for conference rooms that they might want to use for meetings\. During the beginning and end of the week, usage of this application is minimal\. During the middle of the week, more employees are scheduling meetings, so the demand on the application increases significantly\.

The following graph shows how much of the application's capacity is used over the course of a week\.

![\[An example of the capacity demand on an application.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-diagram.png)

Traditionally, there are two ways to plan for these changes in capacity\. The first option is to add enough servers so that the application always has enough capacity to meet demand\. The downside of this option, however, is that there are days in which the application doesn't need this much capacity\. The extra capacity remains unused and, in essence, raises the cost of keeping the application running\.

![\[An example showing how buying more capacity than needed can be inefficient from a cost perspective.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-over-diagram.png)

The second option is to have enough capacity to handle the average demand on the application\. This option is less expensive, because you aren't purchasing equipment that you'll only use occasionally\. However, you risk creating a poor customer experience when the demand on the application exceeds its capacity\.

![\[An example showing how buying less capacity than needed can cause a poor customer experience.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-under-diagram.png)

By adding Amazon EC2 Auto Scaling to this application, you have a third option available\. You can add new instances to the application only when necessary, and terminate them when they're no longer needed\. Because Amazon EC2 Auto Scaling uses EC2 instances, you only have to pay for the instances you use, when you use them\. You now have a cost\-effective architecture that provides the best customer experience while minimizing expenses\.

![\[An example showing how Amazon EC2 Auto Scaling can adjust capacity as needed.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/capacity-example-with-as-diagram.png)

## Example: Web app architecture<a name="autoscaling-design-example"></a>

In a common web app scenario, you run multiple copies of your app simultaneously to cover the volume of your customer traffic\. These multiple copies of your application are hosted on identical EC2 instances \(cloud servers\), each handling customer requests\.

![\[A basic three tier architecture.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sample-3-tier-architecture-diagram.png)

Amazon EC2 Auto Scaling manages the launch and termination of these EC2 instances on your behalf\. You define a set of criteria \(such as an Amazon CloudWatch alarm\) that determines when the Auto Scaling group launches or terminates EC2 instances\. Adding Auto Scaling groups to your network architecture helps make your application more highly available and fault tolerant\.

![\[A basic three tier architecture with an Auto Scaling group.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sample-3-tier-architecture-auto-scaling-diagram.png)

You can create as many Auto Scaling groups as you need\. For example, you can create an Auto Scaling group for each tier\.

To distribute traffic between the instances in your Auto Scaling groups, you can introduce a load balancer into your architecture\. For more information, see [Elastic Load Balancing](autoscaling-load-balancer.md)\.

## Example: Distribute instances across Availability Zones<a name="arch-AutoScalingMultiAZ"></a>

Availability Zones are isolated locations in a given AWS Region\. Each Region has multiple Availability Zones designed to provide high availability for the Region\. Availability Zones are independent, and therefore you increase application availability when you design your application to use multiple zones\. For more information, see [Resilience in Amazon EC2 Auto Scaling](disaster-recovery-resiliency.md)\.

An Availability Zone is identified by the AWS Region code followed by a letter identifier \(for example, `us-east-1a`\)\. If you create your VPC and subnets rather than using the default VPC, you can define one or more subnets in each Availability Zone\. Each subnet must reside entirely within one Availability Zone and cannot span zones\. For more information, see [How Amazon VPC works](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html) in the *Amazon VPC User Guide*\. 

When you create an Auto Scaling group, you must choose the VPC and subnets where you will deploy the Auto Scaling group\. Amazon EC2 Auto Scaling creates your instances in your chosen subnets\. Each instance is thus associated with a specific Availability Zone chosen by Amazon EC2 Auto Scaling\. When instances launch, Amazon EC2 Auto Scaling tries to evenly distribute them between the zones for high availability and reliability\.

The following image shows an overview of multi\-tier architecture deployed across three Availability Zones\.

![\[A typical Auto Scaling group spanning three Availability Zones.\]](http://docs.aws.amazon.com/autoscaling/ec2/userguide/images/sample-3-tier-architecture-with-azs-diagram.png)

### Instance distribution<a name="AutoScalingBehavior.Rebalancing"></a>

Amazon EC2 Auto Scaling automatically tries to maintain equivalent numbers of instances in each enabled Availability Zone\. Amazon EC2 Auto Scaling does this by attempting to launch new instances in the Availability Zone with the fewest instances\. If there are multiple subnets chosen for the Availability Zone, Amazon EC2 Auto Scaling selects a subnet from the Availability Zone at random\. If the attempt fails, however, Amazon EC2 Auto Scaling attempts to launch the instances in another Availability Zone until it succeeds\.

In circumstances where an Availability Zone becomes unhealthy or unavailable, the distribution of instances might become unevenly distributed across the Availability Zones\. When the Availability Zone recovers, Amazon EC2 Auto Scaling automatically rebalances the Auto Scaling group\. It does this by launching instances in the enabled Availability Zones with the fewest instances and terminating instances elsewhere\.

### Rebalancing activities<a name="AutoScalingBehavior.InstanceUsage"></a>

Rebalancing activities fall into two categories: Availability Zone rebalancing and capacity rebalancing\.

**Availability Zone rebalancing**

After certain actions occur, your Auto Scaling group can become unbalanced between Availability Zones\. Amazon EC2 Auto Scaling compensates by rebalancing the Availability Zones\. The following actions can lead to rebalancing activity:
+ You change the Availability Zones associated with your Auto Scaling group\. 
+ You explicitly terminate or detach instances or place instances in standby, and then the group becomes unbalanced\.
+ An Availability Zone that previously had insufficient capacity recovers and now has additional capacity\.
+ An Availability Zone that previously had a Spot price above your maximum price now has a Spot price below your maximum price\.

When rebalancing, Amazon EC2 Auto Scaling launches new instances before terminating the earlier ones\. This way, rebalancing does not compromise the performance or availability of your application\.

Because Amazon EC2 Auto Scaling attempts to launch new instances before terminating the earlier ones, being at or near the specified maximum capacity could impede or completely halt rebalancing activities\.

To avoid this problem, the system can temporarily exceed the specified maximum capacity of a group during a rebalancing activity\. It can do so by a margin of 10 percent or one instance, whichever is greater\. The margin is extended only if the group is at or near maximum capacity and needs rebalancing\. This can happen because of user\-requested rezoning, or to compensate for zone availability issues\. The extension lasts only as long as needed to rebalance the group \(typically a few minutes\)\.

**Capacity Rebalancing**

You can turn on Capacity Rebalancing for your Auto Scaling groups when using Spot Instances\. This lets Amazon EC2 Auto Scaling attempt to launch a Spot Instance whenever Amazon EC2 reports that a Spot Instance is at an elevated risk of interruption\. After launching a new instance, it then terminates an earlier instance\. For more information, see [Use Capacity Rebalancing to handle Amazon EC2 Spot interruptions](ec2-auto-scaling-capacity-rebalancing.md)\.