# Secure Services' Project

## AWS Diagram
![](/attachments/aws-diagram.png)

## AWS Setup

The first step is create a VPC, with IPv4 CIDR Block of `10.0.0.0/16`.

3 Subnets:
- 10.0.0.0/24 - Public Subnet (WAN)
- 10.0.1.0/24 - Private Subnet (DMZ)
- 10.0.2.0/24 - Private Subnet (LAN)

During the creation of the vpc we choose the `Resource to create`, which is `VPC and more` because we need more than the VPC, we need `subnets`, an `Internet Gateway` and `route tables`.

So we define the the CIDR block.
![](/attachments/vpc-create-1.png)

The number of `public subnets` and `private subnets`, and the CIDR blocks for each one.
![](/attachments/vpc-create-2.png)

We don't require `NAT Gateways`, becuase the `PFSenese` is gonna act as `NAT Gateway`. `VPC endopints` is not required too.
![](/attachments/vpc-create-3.png)

At the end this is all the resources that are going be created.
![](/attachments/vpc-create-4.png)

Click on `Create VPC` and should be created.
![Alt text](/attachments/vpc-create-5.png)

Now we need to create 2 `Security Groups` for PFSense, one of them `public` and the last one `private`, where we're gonna to allow or deny traffic to our instance.

`secure-services-pfsense-public-sg`:

This inbound rule will allow to us access via web to PFSense's Website
![Alt text](/attachments/image-3.png)

`secure-services-pfsense-private-sg`:

This inbound rule will allow the all traffic from anywhere, this is because we're gonna handle the policies inside PFSense.
![Alt text](/attachments/image-2.png)

Now we need to create 3 `Network Interfaces` for our PFSense instance, one for WAN, another for DMZ and the last one for LAN.


Here we specify the name, and the subnet associated, and a custom IP which needs to be in the range of the subnet selected.
![Alt text](/attachments/image-4.png)

We select the Security Group.
![Alt text](/attachments/image-5.png)

Once is created disable the option `Change source/destination check`, because PFSense needs all the traffic to route them via NAT.
![Alt text](/attachments/image-6.png)

Repeat these steps for DMZ and LAN interfaces, where both of the share the same Security Group.

Now we're gonna to create an elastic IP and needs to be associated to our public `NIC`.
![Alt text](/attachments/image-13.png) 

Now go to `Route Tables`, for both `private` route tables you need to add a default route to the `Network Card` depending if is the LAN or DMZ.
![Alt text](/attachments/image-7.png)

This is the default route with `Network Card` as target
![Alt text](/attachments/image-8.png)

Now it's time to create our `PFSense` instance.
![Alt text](/attachments/image-9.png)

We used a cust AMI from PFSense
![Alt text](/attachments/image-10.png)

We selected the `t3.small` instance type, because have 3 NICs, one for WAN, another one for DMZ and the last one for LAN.
![Alt text](/attachments/image-11.png)

A Key Pair for login through SSH
![Alt text](/attachments/image-12.png)

At the Network Settings we select our VPC `secure-services-vpc`, and the subnet `secure-services-subnet-public-1-us-east-1a`, the Auto-assing public IP should be disabled, because we have an Elastic IP. Att the Security Group section we select `Select existing security group` option, but don't select any group, leave empty. Then click on advanced settings.
![Alt text](/attachments/image-14.png)

For NIC 1 we selected the interface of the public subnet with the Elastic IP attached.
![Alt text](/attachments/image-15.png)

For NIC 2 and 3 select the NICs that corresponds each other.
![Alt text](/attachments/image-16.png)

![Alt text](/attachments/image-17.png)

At the end, we wrote a `password` env variable, this password is gonna be the password for PFSense, after first login you must have to change the password for security reasons.

![Alt text](/attachments/image-18.png)

This is the Summary. Click on `Lunch instance`
![Alt text](/attachments/image-19.png)

Wait few minutes and the instance should be running.
![Alt text](/attachments/image-20.png)

Open PFSense through HTTPS with the Public IP, then log in.
![Alt text](/attachments/image-21.png)

// Add DNS Server

Follow the wizard and change the Admin Password
![Alt text](/attachments/image-22.png)

Once is ready click on Finish
![Alt text](/attachments/image-23.png)

Go to Interfaces and assign the interfaces: WAN, DMZ and LAN.
![Alt text](/attachments/image-24.png)

For every NIC enable DHCP, because AWS is gonna provide us the IP Adresses assigned to the NICs.
![Alt text](/attachments/image-25.png)

Verify the IP addresses assigned.
![Alt text](/attachments/image-26.png)

Go to Firewall/NAT/Outbound and select the Manual Outbound NAT/
![Alt text](/attachments/image-27.png)


![Alt text](/attachments/image-28.png)// backup

Create this rules for NAT.
![Alt text](/attachments/image-29.png)

Now some ICMP tests from each NIC.
![Alt text](/attachments/image-30.png)
![Alt text](/attachments/image-31.png)
![Alt text](/attachments/image-32.png)

At this point is time to deploy the instances that is gonna be used: server, fileserver and client.

Repeat this steps for every instance.

Name of your server.
![Alt text](/attachments/image-33.png)

Select the OS, we chose Ubuntu.
![Alt text](/attachments/image-34.png)

Assign the corresponding VPC, Subnet and the Private Security Group.
![Alt text](/attachments/image-36.png)

Select the Key Pair.
![Alt text](/attachments/image-35.png)

Click on advanced settings and assing the IP address.
![Alt text](/attachments/image-37.png)

Select the storage.
![Alt text](/attachments/image-38.png)

The summary. Click on Launch Instance
![Alt text](/attachments/image-39.png)

