# Running a Windows deployment 
### Purpose
This scenario considers the provisioning of a sample Windows based instance, accessing it over a public routable IP using remote desktop protocol

## VPC Functional Coverage
| Function | Result | Notes |
| -------- | ------ | ----- |
| VPC | :white_check_mark: | |
| Floating IPv4 | :white_check_mark: | |
| Subnet & BYO Private IP | :white_check_mark: | |
| Dedicated VSI support | :white_check_mark: | |
| Multiple Network Interfaces in VSI | :white_check_mark: | |
| Windows VSI support | :white_check_mark: | |
| Security groups | :white_check_mark: | |
| Multiple Network Interfaces in VSI | :white_check_mark: | |


### Architecture

![](/Diagram05-30-19.png)





### Documented Steps VPC infrastructure

### Prerequisites

1. Install the [IBM Cloud CLI](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started)
2. Have access to a public SSH key.
3. Install the infrastructure-service plugin.  
   `$ ibmcloud plugin install infrastructure-service`

### Login to IBM Cloud

For a federated account use single sign on:  
   `$ ibmcloud login -sso`  
Otherwise use the default login:  
   `$ ibmcloud login`  
If you have an API Key, use --apikey:  
   `$ ibmcloud login --apikey [your API Key]`  

### 1. Create the base VPC
```
ibmcloud is vpc-create WEB-DEMO --default
###### Parameters: Name, default
Creating vpc WEB-DEMO in resource group default under account IBM - Mac's Org as user eariasn@us.ibm.com...
                            
ID                       a15040d1-8e16-4f3c-bcfb-942b3948d07a   
Name                     WEB-DEMO   
Default                  yes   
Default Network ACL      allow-all-network-acl-a15040d1-8e16-4f3c-bcfb-942b3948d07a(6d1b4afd-1ea6-41ec-8538-7c867e78c7c1)   
Default Security Group   -   
Resource Group           (e1d6e82017384baba3ecfb33f3f6a12c)   
Created                  5 seconds ago   
Status                   available 
```
### 2. Create the base subnet
```
ibmcloud is subnet-create WEB a15040d1-8e16-4f3c-bcfb-942b3948d07a us-south-1 --ipv4-cidr-block 10.240.10.0/24
###### Parameters: Name, VPC-ID, Region, IPV4-block
Creating Subnet WEB in resource group default under account IBM - Mac's Org as user eariasn@us.ibm.com...
                    
ID               5f09dce3-72fd-469e-a110-648fcc23cd9f   
Name             WEB   
IPv*             ipv4   
IPv4 CIDR        10.240.10.0/24   
IPv6 CIDR        -   
Addr available   251   
Addr Total       256   
ACL              allow-all-network-acl-a15040d1-8e16-4f3c-bcfb-942b3948d07a(6d1b4afd-1ea6-41ec-8538-7c867e78c7c1)   
Gateway          -   
Created          2 seconds ago   
Status           pending   
Zone             us-south-1   
VPC              WEB-DEMO(a15040d1-8e16-4f3c-bcfb-942b3948d07a)   
Resource Group   -  
```

### Create an SSH Key

An SSH key is required when creating a VPC instance. Copy the ssh public key you wish to use to vpc-key.pub and call the key-create command to load it to the VPC environment. Remember the key id for later use.

```
$ ibmcloud is key-create vpc1-key @vpc-key.pub
Creating key vpc1-key under account Phillip Trent's Account as user pltrent@us.ibm.com...

ID            636f6d70-0000-0001-0000-00000014087d
Name          vpc1-key
Type          rsa
Length        2048
FingerPrint   SHA256:sAu2DF1zXNJQ99XghxZo7DXFXpPFO3PwWjY5a03VIPI
Key           ssh-rsa AAAAB3NzaC1BBBBBByc2EAAAADAQABAAABAQCnbhYSnc8DGQF3A3MR3zLynU4FF8UVVBjnctc3RTeNmWoRny4AJLpI06G9dlmC15QBzDMrNfy0srZnh/YMFlHcN5C73VbLdUJMj0QOqxYSPZgvKKKKrKlBn1WDjigOseO2/NmKIgk3d7lz/iEtkCNlNjNcRPWs3pPkh0NPxIMqsIwvxeWTVsv0OFktKAUA1uXvSFjx4JJRw7hy6tvgJVScbP2Mc2539pxGxiSAMNcqmHFWCQJhwIL2yHJiIcbZ33BDC1BbGg8XReCv0ZVmfXgSs+zuhJb9hDoVCElVDbzXaKs64zMREpy1NUzYQk4o9iahwLXp8gI8qOCzBx pltrent@phillips-mbp.raleigh.ibm.com

Created       1 second ago
```

### 4. Create the base instance for Windows deployment
```
ibmcloud is instance-create WEB01 a15040d1-8e16-4f3c-bcfb-942b3948d07a us-south-1 b-4x16 5f09dce3-72fd-469e-a110-648fcc23cd9f 1000 --image-id cc8debe0-1b30-6e37-2e13-744bfb2a0c11 --key-ids 636f6d70-0000-0001-0000-00000014087d
Creating instance WEB01 in resource group default under account IBM - Mac's Org as user eariasn@us.ibm.com...
###### Parameters: Name, VPC-ID, Region, Flavor, subnet, Network Speed, image-id, key-id

                     
ID                6e792d46-8c90-4288-89ac-4447ad46b2ef   
Name              WEB01   
Profile           b-4x16   
CPU Arch          amd64   
CPU Cores         4   
CPU Frequency     2000   
Memory            16   
Primary Intf      primary(8ba85a22-9f03-4291-8d6a-cddf03de181b)   
Primary Address   10.240.10.13   
Image             centos-7.x-amd64(cc8debe0-1b30-6e37-2e13-744bfb2a0c11)   
Status            pending   
Created           9 seconds ago   
VPC               WEB-DEMO(a15040d1-8e16-4f3c-bcfb-942b3948d07a)   
Zone              us-south-1   
Resource Group    -   
macbook-pro:bin earias
```

### 5. Reserve a floating IP for connectivity
```
ibmcloud is  floating-ip-reserve WEB01-eth0 --zone us-south-1
Creating floating IP WEB01-eth0 in resource group default under account IBM - Mac's Org as user eariasn@us.ibm.com...
###### Parameters: Name, region
                    
ID               e136da39-d05d-49b6-82f4-008c9b3bdc7a   
Address          169.61.244.100   
Name             WEB01-eth0   
Target           -   
Target Type      -   
Target IP           
Created          now   
Status           pending   
Zone             us-south-1   
Resource Group   -   
Tags             -   
```

### 6. Assign the floating IP to the instance that is going to host the web services
```
ibmcloud is instance-network-interface-floating-ip-add 6e792d46-8c90-4288-89ac-4447ad46b2ef 8ba85a22-9f03-4291-8d6a-cddf03de181b e136da39-d05d-49b6-82f4-008c9b3bdc7a 
Creating floatingip e136da39-d05d-49b6-82f4-008c9b3bdc7a for instance 6e792d46-8c90-4288-89ac-4447ad46b2ef under account IBM - Mac's Org as user eariasn@us.ibm.com...
###### Parameters: Instance-id, Instance-interface-id, floating-ip-id

ID               e136da39-d05d-49b6-82f4-008c9b3bdc7a   
Address          169.61.244.100   
Name             WEB01-eth0   
Target           primary(8ba85a22-.)   
Target Type      intf   
Target IP        10.240.10.13   
Created          9 minutes ago   
Status           available   
Zone             us-south-1   
Resource Group   -   
Tags             -   
```

### 7. Check security groups attached to VPC circuit
```
ibmcloud is vpc a15040d1-8e16-4f3c-bcfb-942b3948d07a 
Getting vpc a15040d1-8e16-4f3c-bcfb-942b3948d07a under account IBM - Mac's Org as user eariasn@us.ibm.com...
#### Parameters: VPC-id
                            
ID                       a15040d1-8e16-4f3c-bcfb-942b3948d07a   
Name                     WEB-DEMO   
Default                  yes   
Default Network ACL      allow-all-network-acl-a15040d1-8e16-4f3c-bcfb-942b3948d07a(6d1b4afd-1ea6-41ec-8538-7c867e78c7c1)   
Default Security Group   vertigo-underpaid-expire-remarry-pupil-unsaved(2d364f0a-a870-42c3-a554-000001153485)   
Resource Group           (e1d6e82017384baba3ecfb33f3f6a12c)   
Created                  2 days ago   
Status                   available  
```

### 8. Check current rules for security groups in the VPC (Remote desktop protocol port is 3389)
```
ibmcloud is security-group-rules 2d364f0a-a870-42c3-a554-000001153485
Listing rules of security group 2d364f0a-a870-42c3-a554-000001153485 under account IBM - Mac's Org as user eariasn@us.ibm.com...
ID                                     Direction   IPv*   Protocol                      Remote   
b597cff2-38e8-4e6e-999d-000002259937   inbound     ipv4   tcp Ports:Min=3389,Max=3389   0.0.0.0/0   
b597cff2-38e8-4e6e-999d-000002251885   inbound     ipv4   all                           vertigo-underpaid-expire-remarry-pupil-unsaved(2d364f0a-.)   
b597cff2-38e8-4e6e-999d-000002251797   outbound    ipv4   all

In case that the port is not open, use the following example to open the necessary port:

ibmcloud is security-group-rule-add 2d364f0a-a870-42c3-a554-000001153485 inbound tcp --port-max 23 --port-min 23 --ip-version ipv4 
Creating rule for security group 2d364f0a-a870-42c3-a554-000001153485 under account IBM - Mac's Org as user eariasn@us.ibm.com...

#### Parameters: security-group-id, DIRECTION PROTOCOL, Max destination port, Min destination port, IP version
                          
ID                     b597cff2-38e8-4e6e-999d-000002275001   
Direction              inbound   
IPv*                   ipv4   
Protocol               tcp   
Min Destination Port   23   
Max Destination Port   23   
Remote                 -  

```

### 9. Connect to the instance using Remote Desktop protocol using the floating IP as target. To properly connect the 
* Get encrypted password from UI and saved to a file
* decode it using: 
```
cat UI_PASSWORD_FILE  | base64 --decode > decoded_base64_password_file
```
Decrypt it using the SSH key used to create the instance:
```
 openssl pkeyutl -in decoded_base64_password_file -decrypt -inkey ~/.ssh/id_rsa -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256 > finalpass
 ```
```
cat finalpass
```

### Error Scenarios

### Documentation Provided




<links to documents>
