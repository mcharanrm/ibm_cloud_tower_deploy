Create and Destroy IBM Cloud Instances
=======================================

Tower CPT was broken due to increase in the disk IOPS requirements in Ansible Tower v4.0.0. So it made us to migrate from in-house VMs to IBM cloud VSI.

We believe there are two use cases associated with this repo and these are our primitive attempts to describe it.
1. Looking for virtual cloud instances (OR)
2. Looking for Tower deployment on faster disks

Looking for virtual cloud instances ??
------------------------------------

Install Ansible modules collection for IBM cloud

    ansible-galaxy collection install ibm.cloudcollection  

Install IBM Cloud CLI tool to see the available instance-profiles(m4.large, m4.xlarge), OS-images, and etc...

    curl -sL https://raw.githubusercontent.com/IBM-Cloud/ibm-cloud-developer-tools/master/linux-installer/idt-installer | bash

    ibmcloud plugin install vpc-infrastructure

Make sure you have IBM Cloud API key to authenticate with the IBM Cloud platform.

Create an `IC_API_KEY` env variable

    export IC_API_KEY="<your-api-key>"

Login to IBM cloud and list the available geographical regions under your account

      ibmcloud login

      ibmcloud regions

      ibmcloud target -r jp-tok #point to new targeted region

```
[cmusali@cmusali ibm_cloud_tower_deploy]$ ibmcloud login
API endpoint: https://cloud.ibm.com
Region: jp-tok
Authenticating...
OK

Targeted account Performance-Scale (XXXXXXXXXXXXXXXXX)


API endpoint:      https://cloud.ibm.com   
Region:            jp-tok   
User:              ServiceId-XXXXXXXXXXXXXXXXXXxx   
Account:           Performance-Scale (XXXXXXXXXXXXXXXXXX)   
Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'   
CF API endpoint:      
Org:                  
Space:    

[cmusali@cmusali ibm_cloud_tower_deploy]$ ibmcloud regions
Listing regions...

Name       Display name   
au-syd     Sydney   
in-che     Chennai   
jp-osa     Osaka   
jp-tok     Tokyo   
kr-seo     Seoul   
eu-de      Frankfurt   
eu-gb      London   
ca-tor     Toronto   
us-south   Dallas   
us-east    Washington DC   
br-sao     Sao Paulo   
```

List instance profiles in the region `jp-tok`

    ibmcloud is instance-profiles

```
Listing instance profiles in region jp-tok under account Performance-Scale as user ServiceId-XXXXXXXXXXXXXXXXX...
Name            Architecture   Family     vCPUs   Memory(GiB)   Bandwidth(Mbps)   Volume bandwidth(Mbps)   GPUs   Storage(GB)   
bx2-2x8         amd64          balanced   2       8             4000              1000                     -      -   
bx2d-2x8        amd64          balanced   2       8             4000              1000                     -      1x75   
bx2-4x16        amd64          balanced   4       16            8000              2000                     -      -   
bx2d-4x16       amd64          balanced   4       16            4000              1000                     -      1x150   
bx2-8x32        amd64          balanced   8       32            16000             4000                     -      -   
bx2d-8x32       amd64          balanced   8       32            16000             4000                     -      1x300   
bx2-16x64       amd64          balanced   16      64            32000             8000                     -      -   
bx2d-16x64      amd64          balanced   16      64            32000             8000                     -      1x600   
bx2-32x128      amd64          balanced   32      128           64000             16000                    -      -   
bx2d-32x128     amd64          balanced   32      128           64000             16000                    -      2x600   
```

List all the available images in the region `jp-tok`

    ibmcloud is images


```
Listing images in all resource groups and region jp-tok under account Performance-Scale as user ServiceId-8cd111e5-14e8-4f45-897b-27d440a90012...
ID                                          Name                                               Status       Arch    OS name                              OS version                                File size(GB)   Visibility   Owner type   Encryption   Resource group   
r022-f10e4ea0-1f0b-4fb6-8cfd-1c9aad475046   ibm-centos-7-9-minimal-amd64-3                     available    amd64   centos-7-amd64                       7.x - Minimal Install                     1               public       provider     none         -   
r022-95e7a9a1-8707-49ea-bdef-693311570ce0   ibm-centos-8-3-minimal-amd64-3                     available    amd64   centos-8-amd64                       8.x - Minimal Install                     1               public       provider     none         -   
r022-0fd54b16-2f03-4f8c-8045-38f9781e9071   ibm-debian-10-8-minimal-amd64-1                    available    amd64   debian-10-amd64                      10.x Buster/Stable - Minimal Install      1               public       provider     none         -   
r022-54098fbc-4285-4fc3-aa20-d9da85781fa5   ibm-debian-9-13-minimal-amd64-4                    available    amd64   debian-9-amd64                       9.x Stretch/Stable - Minimal Install      1               public       provider     none         -   
r022-f5387730-7a4b-4f71-9a85-13b05b137953   ibm-redhat-7-6-amd64-sap-applications-1            available    amd64   red-7-amd64-sap-applications         7.x for Applications                      2               public       provider     none         -   
r022-60d279a0-b328-40eb-a379-595ca53bee18   ibm-redhat-7-6-amd64-sap-hana-1                    available    amd64   red-7-amd64-sap-hana                 7.6 for SAP HANA                          2               public       provider     none         -   
r022-71ecd746-b847-4fc4-8144-1ad5ca7095fc   ibm-redhat-7-9-minimal-amd64-3                     available    amd64   red-7-amd64                          7.x - Minimal Install                     1               public       provider     none         -   
```

Launch Plain IBM Cloud Instances
---------------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook create.yml -e ibmcloud_vsi_count=2 \
                                -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                -e install_tower=False
```
TASK [Print IBM Cloud Instance Floating IPs] ***************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "IC instance Floating IP: ",
        [
            "169.63.178.143",
            "169.59.164.19"
        ]
    ]
}
```
```
[cmusali@cmusali ibm_cloud_tower_deploy]$ cat > ic_instance_inventory.ini
[ic_servers]
ic1 ansible_host=169.63.178.143
ic2 ansible_host=169.59.164.19


[ic_servers:vars]
ansible_user = 'root'          
ansible_ssh_private_key_file=conf/towerperf_id_rsa
^C
```

Destroy IBM Cloud Instances
----------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook cleanup.yml -e ibmcloud_vsi_count=2 \
                                 -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                 -e install_tower=False


Looking for Tower deployment on faster disks
---------------------------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook create.yml -e ibmcloud_vsi_count=2 \
                                -e ibmcloud_vpc_name_prefix='perf-scale-test'


Destroy IBM Cloud Instances If Tower is Deployed
------------------------------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook cleanup.yml -e ibmcloud_vsi_count=2 \
                                 -e ibmcloud_vpc_name_prefix='perf-scale-test'


Looking for Tower deployment with Isolated Nodes
------------------------------------------------
    ansible-playbook create.yml -e ibmcloud_vsi_count=2 \
                                -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                -e install_iso=True


Destroy IBM Cloud Instances If Tower is Deployed with Isolated Nodes
--------------------------------------------------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook cleanup.yml -e ibmcloud_vsi_count=2 \
                                 -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                 -e install_iso=True

Looking for Tower/Controller 4.z deployment with/without Execution Nodes
------------------------------------------------------------------------
    ansible-playbook create.yml -e ibmcloud_vsi_count=2 \
                                -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                -e install_execution_node=True

    Defaults:
    1. All targets in the [automationcontroller] have node_type=hybrid
    2. All targets in the [execution_nodes] have node_type=execution

    If you plan to specify different node_type to different targets in the inventory then please specify how many how many tower/controller nodes would you like to have node_type as control and how many as hybrid. How many of execution nodes would you like to have node_type as execution and how many as hop

    -e tower_control_nodes=1 \
    -e tower_hyrid_nodes=2 \       # Sum of tower_control_nodes + tower_hyrid_nodes should always be equal to ibmcloud_vsi_count
    -e tower_execution_nodes=2 \
    -e tower_hop_nodes=1           # Sum of tower_execution_nodes + tower_hop_nodes should always be equal to ibmcloud_vsi_execution_instance_count

    If fails to meet the above requirements then the inventory sections will be created with default node_type

Destroy IBM Cloud Instances If Tower/Controller 4.z deployed with/without Execution Nodes
-----------------------------------------------------------------------------------------

Run the `ansible-playbook` command with following `EXTRA` cmdline arguments

    ansible-playbook cleanup.yml -e ibmcloud_vsi_count=2 \
                                 -e ibmcloud_vpc_name_prefix='perf-scale-test' \
                                 -e install_execution_node=True


To Take Control Over the Default Configuration Values
------------------------------------------------------

Default confing file `vars.yml`

    ibmcloud_vpc_region: 'jp-tok'  # Region for creating IBM cloud instances
    ibmcloud_vpc_name_prefix: 'perf-scale-test'  # Used as prefix when creating various resources
    ibmcloud_vsi_profile: 'bx2-4x16'   # Type of IBM cloud instance, balanced 2 vCPUs and 16 GiB Memory
    ibmcloud_vsi_image_id: 'r022-939bbfe2-d823-4c01-98a3-a6f8193731d8' # OS image ID
    ibmcloud_vsi_count: 3 # Number of required IBM cloud instances
    list_of_ports: [22, 80, 443] # List of ports to open on the IBM cloud instances
