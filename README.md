Ansible Role: AWS EC2 Instances
=========

The ansible role is used to create or delete Amazon EC2 Instances.  When creating instances,
the role provides the ansible facts, private, and public ip addresses in the **files/ec2_facts** folder.

Requirements
------------

1.  Centos 8 Operating System
1.  Python Virtual Environment with the following installed:

    1. **ansible==2.9**
    1. **molecule==3.0.4**
    1. **molecule[docker]**
    1. **boto**
    1. **boto3**

1. An amazon account with the following environment variables set in your **.bashrc** file, or your
   command shell you plan on using:
   
    ```shell script
     AWS_REGION="<your region>"
     AWS_ACCESS_KEY_ID="<your amazon access key id>"
     AWS_SECRET_ACCESS_KEY="<your amazon secret access key id>"
    ```
    
    Replace all text between the double quotes with your Amazon information.

1. Once you pull down the role, create the **files/private\_keys** folder and add the
   private key you plan on using to this directory.  The file name should be the same as
   the name of the private key in Amazon.
   
Role Variables
--------------

1. **defaults/main.yml** variables:

    1. **aws\_vpc** -->
    
        ```yaml
        aws_vpc:
          name: "openshift_dev_vpc"
          security_group: "openshift_dev_vpc_security_group"
          subnets:
            control:
              name: "aws_infrastructure_control_subnet"
        ```
       
       The variable lists the aws vpc the EC2 instances will be created in.
       Let's explain the variable a little more:
       
       1. **name** --> The name of the vpc. The name is needed to look up the VPC id in AWS.
          The VPC id is needed to create or delete the EC2 instances. Note:  The **name**
          corresponds to the **Name** tag on the VPC.  If the tag does not exist, the
          role fails.
       
       1. **security_group** --> The name of the security_group. The security group name is needed to create 
          or delete the EC2 instances.
          
       1. **subnet.control.name** --> The name of the vpc subnet. The name is needed to look up the 
          subnet id in AWS.  The subnet id is needed to create or delete the EC2 instances. Note:  
          The **name** corresponds to the **Name** tag on the subnet.  If the tag does not exist, 
          the role fails. 
          
    1. **ec2\_instances** --> 
       
          ```yaml
          ec2_instances:
            - name: my_instance
              ami: "ami-00594b9c138e6303d"
              instance_type: "t2.medium"
              root_volume_size: 30
              key_name: "my_keypair"
              action: "create"
            - name: your_instance
              ami: "ami-00594b9c138e6303d"
              instance_type: "t2.medium"
              root_volume_size: 25
              key_name: "your_keypair"
              action: "delete"
          ```
          Provides the EC2 Instances you plan on creating or deleting.  Let's explain the variable.
          
          1. The ec2_instances is a list of objects.  In this list, we have two EC2 Instances defined.
           
          1. **name** --> The name of the EC2 instance.  The **name** corresponds to the tag **Name**
            of the EC2 instance.  If you don't have a **Name** tag for the EC2 instance, and you plan
            on deleting the instance, the role will fail.
            
          1. **ami** --> The Amazon AMI id used to create the EC2 instance.  Look on Amazon for the AMI
             you plan on using in your Amazon region.
             
          1. **root_volume_size** --> The Amazon root volume size for the EC2 instance.  This only applies
             for those instances you plan on creating.
          
          1. **instance_type** --> Amazon sizes their EC2 instances.  This is equivalent to the Amazon
             EC2 size.
             
          1. **key_name** --> The Amazon private key file you plan on using.  The file has to be located
          in the **files/private_keys** folder with a permission of 600.  Also, the file name has to be
          the same name as the **key_name** with no extension.
          
          1. **action** --> The action you want to take on the EC2 instance.  Currently, we only have the
             **create** and **delete** action.
          
1. **aws\_region** --> The AWS region you plan on creating or deleting EC2 instances.  In this case,
   we have an environmental variable set, and we pull the value from the variable.

1. **aws\_access\_key** --> The AWS access key you plan on using to create or delete the EC2 instances.
   In this case, we have an environmental variable set, and we pull the value from the variable.

1. **aws\_secret\_key** --> The AWS secret key you plan on using to create or delete the EC2 instances.
   In this case, we have an environmental variable set, and we pull the value from the variable.      
   
Dependencies
------------

None.

Example Playbook
----------------

    ---
    - name: Create EC2 instance playbook
      hosts: localhost
      vars:
        aws_vpc:
          name: "aws_openshift_vpc"
          security_group: "aws_openshift_vpc_security_group"
          subnets:
            control:
              name: "aws_subnet"
        ec2_instances:
          - name: nexus-server
            ami: "ami-00594b9c138e6303d"
            instance_type: "t2.medium"
            root_volume_size: 30
            key_name: "my_keypair"
            action: "create"
        aws_region: "{{ lookup('env', 'AWS_REGION') }}"
        aws_access_key: "{{ lookup('env', 'AWS_ACCESS_KEY_ID') }}"
        aws_secret_key: "{{ lookup('env', 'AWS_SECRET_ACCESS_KEY') }}"
      
      roles:
        - aws-ec2-instances-role


The **vars** should contain the values you are using to override the **default/main.yml**
role variables.  Look above for a description of those variables.

Example Output
--------------

The example output will be in the **aws-ec2-instances-role/files/ec2_facts** folder.

```text
name:  nexus-server
public_ip:  100.25.40.101
private_ip:  10.10.0.10
key_pair:  my_keypair
ssh connection: ssh -i /home/bmullini/Documents/redhat_tools/git/repos/playbooks/roles/aws-ec2-instances-role/files/private_keys/my_keypair centos@100.25.40.101
```


License
-------

BSD

Author Information
------------------

This role was created in 2020 by Bret Mullinix.
