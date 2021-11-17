# CMPE283 : Virtualization
# Assignment 1 - Discovering VMX Features

# Team members
1. Krishnaa Satyanarayana
2. Radhika Jayesh Vyas

# Contributions
1. Krishnaa - <br>
    Setting up the Virtual Machine on Google Cloud with nested virtualization enabled <br>
    Compiling the environment with latest Linux Kernel
   
2. Radhika - <br>
    Compiling default code on the VM <br>
    Identifying MSR capabilities and relevant code changes and recording output

All the work done above was reviewed by each other in order to proceed further.

# Steps 

<h2> Setting up the Virtual Machine on Cloud - </h2>

In the below set of steps we will be setting up a virtual machine on Google cloud with nested virtualisation enabled.

Instance configuration - 

* Image - ubuntu16.04
* Boot disk Size - 200 GB
* CPU Platform - Intel Haswell
* Machine Type - N1 series with 30 GB and 8 vCPU

<b> Instance name - vmubuntu16 </b>

<h2> How to enable nested virtualization on cloud ? </h2>

1. Once the VM instance is up and running on Google Compute Engine click on the <b>"Activate Cloud Shell"</b> on the top right corner
2. Notice that a terminal window corresponding to the project opens at the bottom
3. Execute the following set of commands
     <br> <i> "gcloud compute instances export vmubuntu16   --destination=temp.yaml   --zone=us-central1-a "</i>
   Modify the instance name and zone as per your VM info
4. Open the temp.yaml file which is created in the current directory using vi
5. Update the value for enableNestedVirtualization or add entirely as below 
      <br> <i> "advancedMachineFeatures:
            enableNestedVirtualization: true "</i>
6. Update the VM with the new configuration with the following command 
      <br> <i> "gcloud compute instances update-from-file vmubuntu16   --source=temp.yaml   --most-disruptive-allowed-action=RESTART   --zone=us-central1-a" </i>
7. Now that the configuration is updated into the VM, ssh into the VM and look for if VMX features enabled or not with the below command 
      <br> <i> "grep -cw vmx /proc/cpuinfo" </i>
   A value other than 0 would mean that the VMX features are enabled on the system. 

For further details or troubleshooting guidelines the following blog can be referred. 
https://cloud.google.com/compute/docs/instances/nested-virtualization/overview 

<h2> Setting up the VM with the latest linux kernel </h2>

1. Fork the following repository into your personal github account <br><i> "https://github.com/torvalds/linux" </i> 
2. SSH into the VM using ssh-keygen or the gcloud shell
3. Clone the repository <br><i> "git clone https://github.com/krishna201294/linux.git" </i>
4. Install the following packages and dependencies 
    a. sudo apt-get install make
    b. sudo apt-get install gcc
    c. sudo apt-get install libncurses-dev
    d. sudo apt-get install flex
    e. sudo apt-get install bison
5. Verify the OS and the hardware using the command <i> "uname -a" </i>
6. Copy the config file corresponding to your linux os from the following path into a file called ".config" 
      <br><i> "cp /boot/config-5.11.0-1021-gcp .config"</i>
7. Under the .config file modify the CONFIG_SYSTEM_TRUSTED_KEYS variable value to CONFIG_SYSTEM_TRUSTED_KEYS=""
8. Compile the oldconfig using the following command 
      <br><i> make oldconfig </i>
9. Install the below packages
    a. sudo apt-get install openssl
    b. sudo apt-get install libssl-dev
    c. sudo apt-get install libelf-dev
10. Execute <i> "make prepare" </i>
11. Verify the current version of kernel using the command "uname -r" ( Currently set to 5.11.0 ) 
12. Build the modules directory (8 corresponds to the vCPU's in the VM)
      <br> <i> "make -j 8 modules" </i>
13. Build the entire directory 
      <br> <i> "make -j 8" </i>
14. Install the module packages
      <br> <i> "sudo make INSTALL_MOD_STRIP=1 modules_install" </i> 
15. Install the module
      <br> <i> "sudo make install" </i>
16. Reboot the VM using the following command and reconnect using SSH
      <br> <i> "sudo reboot" </i>
17. Verify the current version of kernel using the command "uname -r" (Notice it is upgraded to 5.15.0+)


<h2> Exploring the VMX Features </h2>

1. Under the linux directory create the following folder structure  " ~/linux/cmpe283/assignment1 "
2. Copy the cmpe283-1.c and Makefile into the folder
3. Make necessary changes in the C code in order to explore VMX features for
    a. VMX_PINBASED_CTLS
    b. VMX_PROCBASED_CTLS
    c. VMX_PROCBASED_CTLS2
    d. VMX_EXIT_CTLS
    e. VMX_ENTRY_CTLS
4. Compile the C file using "make" command
5. Verify if a kernel module file is generated like filename.ko ("cmpe283-1.ko")
6. Load the module into the kernel using the insmod command 
      "<br> <i> sudo insmod cmpe283-1.ko </i>"
7. Verify that the kernel object file is loaded or not. command - lsmod | grep cmpe283
8. The output for executing in the message buffer. command
      "<br> <i> dmesg </i>"
9. Sample output starts to look as below 
      "<br><i> [ 2990.034812] cmpe283_1: loading out-of-tree module taints kernel.
      [ 2990.034847] cmpe283_1: module verification failed: signature and/or required key missing - tainting kernel
      [ 2990.036497] CMPE 283 Assignment 1 Module Start
      [ 2990.036502] Pinbased Controls MSR 
      </i>"
10. The module can be unloaded using the following command if any changes to c file are to be made and reloaded into the kernel
    <br><i> sudo rmmod cmpe283_1 </i>

The <b>output</b> of the file executed in this lab can be found at the following below path

<b> https://github.com/krishna201294/linux/blob/master/cmpe283/assignment1/output.txt </b>
  
