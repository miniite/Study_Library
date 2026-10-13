---
type: "practice"
difficulty: "Intermediate"
---

# How would you connect to a Linux instance in a private subnet that has no public IP address?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Private Endpoints and Network Paths](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Private_Endpoints_and_Network_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To connect to a Linux instance in a private subnet without a public IP address, we need to leverage network configurations such as NAT gateways or direct connections via a bastion host. The core idea is to ensure secure and controlled access while maintaining the security boundary defined by your VPC.

This question is probing your understanding of cloud networking concepts, specifically how to manage connectivity in a private subnet environment. Key assumptions include the use of AWS but similar solutions can be applied to other cloud providers like Azure or GCP. The interviewer might also be looking for an explanation on security best practices and potential limitations.

First-principles reasoning involves starting with the basic principle that the instance cannot directly receive inbound traffic due to its private subnet configuration. We need to establish a connection through an intermediary service.

Using a NAT Gateway:

A NAT gateway can be used to assign public IP addresses for instances in the private subnet.
The NAT gateway will handle all outbound internet requests from the instances and route inbound traffic back out using the public IP of the gateway.
For SSH connections, you would need to allow access through a security group associated with the NAT gateway.

Using an Elastic IP (EIP) and Security Groups:

Assign an EIP to a public-facing instance (like a jump server or bastion host).
Use security groups to control inbound and outbound traffic.
SSH into the bastion host first, then from there SSH into the target instance.

Direct Connect via a Bastion Host:

Set up a bastion host in a public subnet.
Configure security groups to allow only necessary traffic.
Use tools like ssh or terminal servers to securely connect to the bastion host, which then forwards the SSH session to the target instance.

Automated SSH Key Management:

Ensure proper key management for automated deployments and maintenance.
Use IAM roles or policies to restrict access based on least privilege principles.

#### Closing summary
The chosen method should balance security with ease of use. NAT gateways provide a managed solution but add complexity, while bastion hosts offer more control over session logging and monitoring at the cost of additional infrastructure management.

#### Common mistakes/follow-ups
include:

Failing to properly configure security groups.
Not considering long-term operational overhead, such as managing EIPs for every instance.
Overlooking the importance of key management.
Not discussing best practices like least privilege access.

**Comparison:**

```
Method	Description	Pros	Cons
NAT Gateway	Assign public IP to instances in private subnet	Managed, easy setup	Adds complexity, costs
EIP and Security Groups	Use a bastion host with EIP for SSH access	Fine-grained control, secure	Additional management overhead
```

**Code:**

```python
# Example code snippet for setting up a bastion host and SSH connection
import boto3

ec2 = boto3.resource('ec2')

# Create a new EC2 instance as a bastion host in the public subnet
public_subnet_id = 'subnet-12345678'
image_id = 'ami-abcd1234'

bastion_instance = ec2.create_instances(
    ImageId=image_id,
    MinCount=1,
    MaxCount=1,
    InstanceType='t2.micro',
    SubnetId=public_subnet_id
)[0]

# Add a security group rule to allow SSH access from the internet (beware of overexposure)
sg = ec2.SecurityGroup(bastion_instance.security_groups[0].group_id)
sg.authorize_ingress(
    IpProtocol='tcp',
    FromPort=22,
    ToPort=22,
    CidrIp='0.0.0.0/0'  # This should be a restricted CIDR block
)

# SSH into the bastion host and then to the target instance
bastion_public_ip = bastion_instance.public_ip_address

ssh_command = f"ssh -A ec2-user@{bastion_public_ip} 'ssh target_user@192.168.0.5'"

print(ssh_command)
```

**Example input / output:**

```text
# Example input to create a new bastion host instance
$ python setup_bastion_host.py

# Example output showing the command to SSH into the target instance via the bastion host
ssh -A ec2-user@1.2.3.4 'ssh target_user@192.168.0.5'
```

**Data:**

```json
{
  "subnet_id": "subnet-12345678",
  "image_id": "ami-abcd1234",
  "instance_type": "t2.micro"
}
```

**Flow:**

1. SSH Flow
2. Create Bastion Host
3. Configure Security Group
4. Autowise SSH Session
5. Connect to Target Instance
6. SSH -A ec2-user@bastion_host'sshtarget_user@target_instance'

