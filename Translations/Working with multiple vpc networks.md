## Working with multiple VPC networks Using the command line only

## OBJECTIVES

*Create custom mode VPC network with firewall rules

*Create VM instances

*Explore the connectivity between VM instances

*Create a VM instance with multiple network interfaces

Activate the google cloud shell

## Objective one
Create custom mode VPC networks with firewall rules

Step 0ne
Create two custom networks, managementnet and privatenet, along with firewall rules to allow SSH, ICMP, and RDP ingress traffic.
By default,every google project is configured with a default network and a mynetwork network has being preconfigured for the lab


#Create the managementnet network
   Create the network with the following

*Name - managementnet
*For subnet creation mode, Use custom mode
*Use managementsubnet-us as the subnet for the managementnet
*Use the us-central1 region
*Use this IP address range 10.130.0.0/20
*Note down your project_ID
*When the google cloud shell is activated and it is not associated with your project_ID . Use the gcloud config set project project_ID command.

Input the command to create the managementnet
*gcloud compute networks create managementnet --project=PROJECT_ID --subnet-mode=custom --bgp-routing-mode=regional
Input the command to create the managementsubnet
*gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-00-bde28d3609a5 --range=10.130.0.0/20 --network=managementnet --region=us-central1

Create the privatenet network
*gcloud compute networks create privatenet --subnet-mode=custom

Create the privatesubnet-us
*gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

Create the privatesubnet-eu
*gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

To verify the avaliable VPC network
*gcloud compute networks list

Step two
Create the firewall rules for managementnet
 Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.
 The following parameters are included in the command to create the firewall
 *Name- managementnet-allow-icmp-ssh-rdp
 *Network - managementnet
 *Targets - All instances in the network
 *Source filter - IP ranges
 *Source IP ranges - 0.0.0.0/0
 *protocols and ports - specified protocols and ports
  For protocol ,use tcp ,ICMP
  For ports ,use 22 and 3389

Input the command to create the firewall with the above parameters
  *gcloud compute --project=qwiklabs-gcp-00-bde28d3609a5 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0

Create the firewall rules for privatenet
*gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

Verify the firewall rules by VPC network
*gcloud compute firewall-rules list --sort-by=NETWORK

## Objective two
   Create two VM instances in the managementsubnet and  privatesubnet

Create the management-us-vm with the following parameters
-Name - managementnet-us-vm
-Use the us-central1 region
-Use the us-central1-c as the zone
-Machine type - 1vcpu(3.75GB memory,n1-standard-1)
-Network interfaces
 *Network-managementnet
 *Subnetwork - managementsubnet-us

 Input the command to create the managementnet-us-vm
 *gcloud compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=e2-medium --subnet=managementsubnet-us --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=managementnet-us-vm --reservation-affinity=any

Create the privatenet-us-vm instance
 *gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us

Verify the list of all VM instances by zone
  *gcloud compute instances list --sort-by=ZONE

## Objective three
Explore the connectivity between the VM in instances

Step one
Use the ping command to confirm VM instances can reach each other over the network
  - To check the external IP addresses of the following VM instances, Use the command below
  *gcloud compute instances list --sort-by=ZONE
  *Note the external IP addresses of the mynet-eu-vm, managementnet-us-vm, privatenet-us-vm
Connect to the mynet-us-vm through SSH

*gcloud compute ssh mynet-us-vm

Ping mynet-eu-vm external IP address and internal address from mynet-us-vm

*ping -c 3 <Enter mynet-eu-vm's external IP here>
*ping -c 3 <Enter mynet-eu-vm's internal IP here>

Ping managementnet-us-vm external IP address and internal IP address

*ping -c 3 <Enter managementnet-us-vm's external IP here>
*ping -c 3 <Enter managementnet-us-vm's internal IP here>

Ping privatenet-us-vm external IP address and internal IP address
*ping -c 3 <Enter managementnet-us-vm's external IP here>
*ping -c 3 <Enter privatenet-us-vm's internal IP here>

## Objective Four
Create a VM instance with multiple network interfaces with the following parameters

-Name : vm-appliance
-Region: us-central1
-zone: us-central1-c
-Machine type: 4vCPUs(15 GB memory,n1-standard-4)
-Network interfaces
 Network : privatesubnet
 Subnetwork: privatesubnet-us
 Network : managementnet
 Subnetwork: managementsubnet-us
 Network: mynetwork
 Subnetwork: mynetwork

 Input the command to create the VM instance with multiple interfaces
 *gcloud compute instances create vm-appliance --region=us-central1 --zone=us-central1-c --machine-type=n1-standard-4 --network=privatenet --subnet=privatesubnet-us --network=managementnet --subnet=managementsubnet-us --network=mynetwork --subnet=mynetwork

 Explore the network interface details of vm-appliance through the VM's terminal
 -Connect to the vm-appliance through SSH

*gcloud compute ssh vm-appliance

 - To list the network interfaces of the vm-appliance VM.
 *sudo ifconfig

 Explore connectivity from the vm-appliance to the mynetwork ,managementsubnet-us and privatesubnet subnet networks due to its multiple interfaces

 Connect to the vm-appliance thrught SSH
 *gcloud compute ssh vm-appliance

Ping to test connectivity to the internal IP's of the mynet-eu-vm, mynet-us-vm, privatenet-us-vm and managementnet-us-vm VM instances respectively
*ping -c 3 <Enter mynet-eu-vm's internal IP here>
*ping -c 3 <Enter mynet-us-vm's internal IP here>
*ping -c 3 <Enter privatenet-us-vm's internal IP here>
*ping -c 3 <Enter managementnet-us-vm's internal IP here>

To list the routes for vm-appliance instance
* ip route
