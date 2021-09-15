# Cisco Modeling Labs on AWS


## Project Overview
Instructions to deploy the Cisco Modeling Labs (CML) network simulation tool on AWS.


## Solution Components
- CML Personal ($199/year)
- VMware Workstation (to convert the `ova` file into a `vmdk` file)
- AWS: EC2, S3 (to store the `vmdk` file which becomes an AMI)


## Usage
- **GUI** - HTTPS to the EC2's external IP address
- **CLI** - SSH to the EC2's external IP address
  - CLI directly to the lab devices: SecureCRT > Properties > Connection > Logon Actions > Remote command: `open /lab_name/node_id/line_#`
    

## Build
### Laptop
- Download the `OVA` and `ISO` files and the license token from [Cisco](https://learningnetworkstore.cisco.com/myaccount)
- Open the `OVA` file in VMware Workstation
  - Networking Adapter: Bridged (may need to specify the NIC in the Virtual Network Editor)
  - mount the `refplat ISO` as a CD/DVD
  - power up the VM and configure `admin` and `sysadmin` accounts
  - power down the VM and export it: File > export to `OVF`
- Extract the `ISO` image to get the device image `qcow2` files
- Upload the `.vmdk` file to S3
- [Import the VM into AWS as an AMI image](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html)
  ```
  aws iam create-role --role-name vmimport --assume-role-policy-document "file://C:\Users\gdavitiani\Desktop\trust-policy.json"
  aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file://C:\Users\gdavitiani\Desktop\role-policy.json"
  aws ec2 import-image --description "Cisco CML" --disk-containers "file://C:\Users\gdavitiani\Desktop\containers.json"
  
  aws ec2 describe-import-image-tasks --import-task-ids import-ami-0143a066d6e195d3b
  ```
### AWS Console
- Launch an EC2 instance from the newly created image
  - instance type: `c5n.metal`
  - create and assign an elastic public IP address

### Browser
- **GUI** to CML: HTTPS to the instance's external IP address or DNS name
- Tools > Licensing > Register: `<TOKEN>`
- Node and image **definition** `yaml` files get imported with the VM, but the **image** `qcow2` files need to be uploaded
- Tools > Node and Image Definitions > Image Definitions > Manage > `FILENAME.qcow2` > Upload Image

### AWS Console
- **CLI** to CML: AWS Console > Instances > Connect > Session Manager > Connect
- Copy the uploaded images to their corresponding folders
  ```
  su sysadmin
  sudo find / -name *.qcow2
  
  sudo cp /var/local/virl2/dropfolder/csr1000v-universalk9.17.03.01a-serial.qcow2 /var/lib/libvirt/images/virl-base-images/csr1000v-170301a
  sudo cp /var/local/virl2/dropfolder/vios_l2-adventerprisek9-m.ssa.high_iron_20190423.qcow2 /var/lib/libvirt/images/virl-base-images/iosvl2-2019
  sudo cp /var/local/virl2/dropfolder/vios-adventerprisek9-m.spa.159-3.m2.qcow2 /var/lib/libvirt/images/virl-base-images/iosv-159-3
  sudo cp /var/local/virl2/dropfolder/alpine-3-12-base.qcow2 /var/lib/libvirt/images/virl-base-images/alpine-3-12-base
  ```
  
- Increase the VM's storage capacity (optional)
  ```
  sudo yum install cloud-utils-growpart
  
  # show existing volume sizes and names
  df -hT
  lsblk
  sudo pvscan
  sudo vgdisplay
  sudo lvdisplay

  # resize the partition
  sudo growpart /dev/nvme0n1 2
  
  # resize the physical volume
  sudo pvresize /dev/nvme0n1p2

  # resize the logical volume
  sudo lvextend -L+32G /dev/cl_cml2-controller/root

  # resize the file system
  sudo xfs_growfs -d /

  # verify
  df -hT
  lsblk
  ```


## Docs
- [CML Product Page](https://www.cisco.com/c/en/us/products/cloud-systems-management/modeling-labs/index.html)
- [CML Documentation](https://developer.cisco.com/docs/modeling-labs/)
- [CML Support Forum](https://learningnetwork.cisco.com/s/topic/0TO3i00000094ZjGAI/cisco-modeling-labs-personal-community)
