# EC2 Lost Key Recover

In this workshop you will be able to recover ssh access to an Amazon EC2 instance if the required private key pair has been lost.

The application uses [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), [Amazon EC2](https://aws.amazon.com/ec2/), [AWS Systems Manager](https://aws.amazon.com/systems-manager/) and the [Amazon CLI](https://aws.amazon.com/cli/).

## Prerequisites

### AWS Account

In order to complete this workshop you will need an AWS Account with access to create AWS IAM, and EC2 recorces. The code and instructions in this workshop assume only one student is using a given AWS account at a time and to a user that has CLI access. If you try sharing an account with another student, you'll run into naming conflicts for certain resources. You can work around these by appending a unique suffix to the resources that fail to create due to conflicts, but the instructions do not provide details on the changes required to make this work.

All of the resources you will launch as part of this workshop are eligible for the AWS free tier if your account is less than 12 months old. See the [AWS Free Tier page](https://aws.amazon.com/free/) for more details.


### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- AWS Systems Manager
- Amazon EC2

You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services.


## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

### 1. Create an IAM Role.

Create an IAM Role that will be later attached to an EC2 instance.  This role will allow an EC2 instance to have access to Systems Manager.



<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>
1. From the AWS Management Console, click on **Services** and then select **IAM** in the Security, Identity & Compliance section.

1. Select **Roles** in the left navigation bar and then choose **Create new role**.

1. Select **EC2** as service to use the role. Click **Next: Permissions**.

1. Type **SSM** in the search bar and check the box left of **AmazonEC2RoleforSSM**. click **Next: Review**.


1. Give it the role name `AmazonEC2RoleforSSMRole` and click **Create Role**.
</p></details>

---

### 2. Create an EC2 instance with no key-pair attached.

Start an Amazon EC2 and start without assiginging a keypair.




<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. From the AWS Management Console, click on **Services** and then select **EC2** in the Compute section.

1. Click on **Launch Instance**.

1. Choose an Amazon Machine Image (AMI), for this example I choose **Amazon Linux 2 AMI (HVM), SSD Volume Type**.  Click **Select**.

1. Choose an Instance Type, for this example **t2.micro** will work.  Click **Next: Configure Instance Details**.

1. Configure Instance Details, under **IAM role** select the created role from **Step 1** which was  `AmazonEC2RoleforSSMRole`.  Click **Next: Add Storage**.

1. Add Storage, the default is fine.  Click **Next: Add Tags**.

1. Click **Add Tag** and for *Key* put `Name`.  For *Value* put `EC2noKeyPair`.  Click **Next: Configure Security Group**.

1. Configure Security Group, use an existing security group or create a new one.  Ensure that the EC2 instance has access to **Port 22**.  Click **Review and Launch**.

1. Review Instance Launch, when complete.  Click **Launch**.

1. A pop up will appear to set an EC2 Key Pair.  Select **Proceed without a key pair** and **check the box** to acknologe that you will not be able to connect to the instance unless you know the password built into the AMI.

</p></details>

---

### 3. Install OpenSSL

OpenSSL is a robust, commercial-grade, and full-featured toolkit for the Transport Layer Security (TLS) and Secure Sockets Layer (SSL) protocols and will be used to generate a key pair.  If you have OpenSSL installed then this step can be skipped.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Download [OpenSSL](https://www.openssl.org/) onto your local computer.  For this workshop the directions will be made to be followed on a **mac**.

1. If **brew** is installed then enter the following command into a termianl.  If you do not have brew installed then install it from this [link](https://brew.sh/).

	``
	brew install openssl
	``

1. In the terminal enter the bellow command to verify that *OpenSSL version 1.0* or newer is installed.

	``
    openssl version
    ``

</p></details>

---

### 4. Create a key-pair using OpenSSL.

In this section you'll generate a key pair.


<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the terminal enter the bellow command.  The name for **private_key.pem** can be changed.

	``
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out private_key.pem
    ``

1. Verify that the private key has been generated and move to the next step.

</p></details>

---

### 5. Generate a Systems Manager Command.

The Systems Manager gives an output that can run the command via the terminal and AWS CLI.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. From the AWS Management Console, click on **Services** and then select **EC2** in the Compute section.

1. On the left hand side, expand the **SYSTEMS MANAGER SERVICES** section and click on **Run Command**.

1. Click **Run Command**.

1. In the **Command document** section choose **Command document**.

1. In the **Select Targets by** section select the EC2 instance created from step 2.  If there are now instances then the **SSM Agent** might not be installed on the EC2.

1. Go down to the bottom of the page and expand the **AWS Command Line Interface command** section.

1. **Copy** the contents in the **CLI command** text block and save to be used in the next steps.

1. Change the `"commands":[""]` section to `"commands":["echo \"\">> /home/ec2-user/.ssh/authorized_keys "]`
 
</p></details>

---

### 6. Sending SSM command to EC2 instance.

In this module you'll send a Systems Manager command to the ec2 isntance via the terminal and Amazon CLI.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the terminal, go to the directory to where the private key was made and change the permissions of the key by using the below command.

	``
    chmod 400 private_key.pem
    ``

1. Once the permisions are changed then run the below command and copy the output.

	``
	ssh-keygen -y -f private_key.pem
    ``

1. From the Systems Manager command in the `echo \"\"` part.  Paste the ssh key output between the quotes.  Press **enter** to send the command.

</p></details>

---

### 7. SSH onto the EC2 instance.

In this module you'll SSH into the EC2 instance.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. From the AWS Management Console, click on **Services** and then select **EC2** in the Compute section.

1. Copy the **Public DNS (IPv4)** from the running EC2 instance.

1. In the terminal enter the command `ssh -i "private_key.pem" ec2-user@` then paste the **Public DNS (IPv4)** to the end of the command. Press **enter** and below you should see...

	```
       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|
	```

</p></details>


---

## Finished
You now ssh'd into an instance that was created with no key pair!

