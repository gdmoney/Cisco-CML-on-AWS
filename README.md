# Cisco Modeling Labs on AWS


## Project Overview
Instructions to deploy the Cisco Modeling Labs (CML) network simulation tool on AWS.


## Solution Components
- CML Personal ($199/year)
- VMware Workstation
- AWS: EC2, S3


## Usage
HTTPS to the ec2's public IP address


## Build
- download the OVA and ISO files and the license token from [Cisco](https://learningnetworkstore.cisco.com/myaccount)
- open the OVA file in VMware workstation
  - Networking Adapter: Bridged (may need to specify the NIC in the Virtual Network Editor)
  - mount the refplat ISO as a CD/DVD
  - power up the VM and configure `admin` and `sysadmin` accounts
  - power down the VM > File > export to OVF
- upload the `.vmdk` file to S3
- [import the VM in AWS as an AMI image](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html)
```
aws iam create-role --role-name vmimport --assume-role-policy-document "file://C:\Users\gdavitiani\Desktop\trust-policy.json"
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file://C:\Users\gdavitiani\Desktop\role-policy.json"
aws ec2 import-image --description "Cisco CML" --disk-containers "file://C:\Users\gdavitiani\Desktop\containers.json"

aws ec2 describe-import-image-tasks --import-task-ids import-ami-0143a066d6e195d3b
```

- launch an instance from the image
  - instance type: `c5n.metal`
  - create and assign an elastic public IP address
- **GUI** to the CML: https to the instance's external IP address
- Tools > Licensing > Register: `<TOKEN>`
- node and image definition `.yaml` files get imported with the VM but the image `.qcow2` files need to be uploaded
- Tools > Node and Image Definitions > Image Definitions > Manage > `FILENAME.qcow2` > Upload Image
- **CLI** to the CML: AWS Console > Instances > Connect > Session Manager > Connect
- copy the uploaded image(s) into their corresponding folder(s)
```
su sysadmin
sudo find / -name *.qcow2
sudo cp /var/local/virl2/dropfolder/csr1000v-universalk9.17.03.01a-serial.qcow2 /var/lib/libvirt/images/virl-base-images/csr1000v-170301a
sudo cp /var/local/virl2/dropfolder/vios_l2-adventerprisek9-m.ssa.high_iron_20190423.qcow2 /var/lib/libvirt/images/virl-base-images/iosvl2-2019
sudo cp /var/local/virl2/dropfolder/vios-adventerprisek9-m.spa.159-3.m2.qcow2 /var/lib/libvirt/images/virl-base-images/iosv-159-3
```
  
- increase the VM's storage capacity (optional)
```
sudo yum install cloud-utils-growpart
df -hT
lsblk
sudo growpart /dev/nvme0n1 2
```

**TODO:**
- [x] resize the partition
- [ ] resize the volume group (`cl_cml2-controller`) and the logical volume (`/dev/cl_cml2-controller/root`)
```
sudo vgdisplay
sudo lvdisplay
sudo pvscan
```
- [ ] external connectivity directly to the networking devices
- [ ] GitHub Actions: cloud or self-hosted
- [ ] TFTP server
- [ ] network automation server
- [ ] configure DNS
