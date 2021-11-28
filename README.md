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




# Assignment 2 - Instrumentation Via Hypercall

# Contributions
1. Krishnaa - <br>
    Investigated on setting up nested VM on Google compute engine and debugging and fixing script changes
   
2. Radhika - <br>
    Investigated on the libraries to identify and store timestamp counters and increment counters

All the work done above was reviewed by each other in order to proceed further.

<h2>Steps</h2>

1. Took up 1st Question: For CPUID leaf node %eax=0x4FFFFFFF:
	Return the total number of exits (all types) in %eax


2. Defined a variable total_exits of type u64 in cpuid.c and exported it in vmx.c.
   Checked condition if eax == 0x4fffffffff, then store the total_exits in %eax.
   In the vmx.c, each time the above condition is true, the value of total_exits is incremented by 1.

3. Took up 2nd:  (For CPUID leaf node %eax=0x4FFFFFFE:
   Return the high 32 bits of the total time spent processing all exits in %ebx
   Return the low 32 bits of the total time spent processing all exits in %ecx 
	%ebx and %ecx return values are measured in processor cycles, across all VCPUs) :
   In the vmx.c file in arch/x86/kvm/vmx/, we made changes to the __vmx_handle_exit function.

4. Defined an exit_handler_index of type u16 in vmx.c which takes up the input of the number and stores it in %ecx.
   Also defined total_cycle_time of the type atomic64_t in cpuid.c and exported it to vmx.c.
   Defined 2 variables, start_time and end_time and stored the value of rdtsc() in start time.
   In the function atomic_fetch_add(), passed the parameter of delta(end_time - start_time) and stored it in total_cycle_time.
   In cpuid.c, we checked the condition if eax == 0x4fffffffe, then we shift the high 32-bits in %ebx and low 32 bits in %ecx of total_time.

5. Now, we are required to make a VM. In order to get gcloud on our desktop, we need to install packages defined below using the following commands :

    <br>$ sudo apt-get update
    <br>$ sudo apt-get upgrade
    <br>$ sudo apt-get install gnome-shell
    <br>$ sudo apt-get install ubuntu-gnome-desktop
    <br>$ sudo apt-get install autocutsel
    <br>$ sudo apt-get install gnome-core
    <br>$ sudo apt-get install gnome-panel
    <br>$ sudo apt-get install gnome-themes-standard


6. Once done with all the installation, we installed tightvncserver using the below commands:
    <br>$ sudo apt-get install tightvncserver
    <br>$ touch ~/.Xresources
    <br>Then launch the server using “$ tightvncserver” command.

7. After setting up password for the Ubuntu Desktop Virtual Machine, start up the code using the command :

    <br>:~$ vim /home/username/.vnc/xstartup

8. After the startup, we need to write a startup script, which is as follows :

    <br>#!/bin/sh
    <br>autocutsel -fork
    <br>xrdb $HOME/.Xresources
    <br>xsetroot -solid grey
    <br>export XKL_XMODMAP_DISABLE=1
    <br>export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity"
    <br>export XDG_MENU_PREFIX="gnome-flashback-"
    <br>unset DBUS_SESSION_BUS_ADDRESS
    <br>gnome-session --session=gnome-flashback-metacity --disable-acceleration-check --debug &

9. Kill the process using ‘$ vncserver -kill :1’ and restart the server ‘$ vncserver -geometry 1024x640’

10. Install google cloud SDK using this link.

11. Once google cloud sdk is installed, we wrote the below command on the local terminal to connect to your current Google cloud instance :

    <br> $ gcloud compute ssh \
    <br>YOUR INSTANCE NAME HERE \
    <br>--project YOUR PROJECT NAME HERE \
    <br>--zone YOUR TIMEZONE HERE \
    <br>--ssh-flag "-L 5901:localhost:5901"

12. Using any remote desktop tool we can connect now to the vm using the public ip

13. Now in the host VM in the remote desktop, we need to install the following packages and execute the commands.

	<br>sudo apt-get update
	<br>sudo apt-get install -y xrdp
	<br>sudo apt-get install -y xfce4
	<br>sudo service xrdp restart


14. After executing the above commands, the host VM is eligible for nested VM. We installed a nested VM with ubuntu 20.04 using the live-server disk image -ubuntu-20.04.3-live-server-amd64.iso

    <br>Install cpuid package in the nested VM using the command 
    <br>$ apt install cpuid
    
    <img width="1436" alt="nestedVM1" src="https://user-images.githubusercontent.com/78173506/142967996-cf26e682-c049-4233-9169-56b06660f852.png">
    
    <img width="1247" alt="nestedVM2" src="https://user-images.githubusercontent.com/78173506/142968166-fe519853-b400-4fdd-b408-3e0fd3d602aa.png">




15. We then ran our test script in order to check our code. The output shows :

    <br>**The total number of exits after each execution.**
    
    <img width="1257" alt="totalexits" src="https://user-images.githubusercontent.com/78173506/142967819-832a6ade-85dc-450f-989e-93b7462ae63e.png">


    <br>**The total time after each execution for exits.**
    
    <img width="516" alt="totalcycletime" src="https://user-images.githubusercontent.com/78173506/142967871-2979865a-2d6f-417d-8d34-9cdf79135646.png">


    <br>**The register output if we use the command “cpuid -l 0x4fffffff”** is shown below:

    <img width="1306" alt="registeroutput" src="https://user-images.githubusercontent.com/78173506/142967903-e9d2afe5-193a-4d5a-ad5c-6143cfb6fe5f.png">




# Assignment 3 - Instrumentation via Hypercall

# Contributions
1. Krishnaa - <br>
    Made the necessary code changes for the total number of exits and the total time cycle for each exit. <br>
    Added the required conditions for SDM and KVM exit codes.
2. Radhika - <br>
    Figured out the codes for which SDM and KVM gives out exits.
    Recorded output for all the questions.

All the work done above was reviewed by each other in order to proceed further and we debugged and traiaged issues in the process. We worked together in building the test script and making the below observations

# Steps followed to make code changes
1. Make necessary code changes in Cpuid.c or vmx.
2. Compiled the code using the following command 
	<br> <i> "make -j 8 modules" </i>
3. Install the module packages <br> <i> "sudo make INSTALL_MOD_STRIP=1 modules_install" </i> 
4. Stop the nested VM which is currently running
5. On the host VM run the following commands
      <br> <i> "rmmod kvm_intel" </i>
      <br> <i> "rmmod kvm" </i>
   This command removes the old binaries of the kvm
6. Install the latest binaries using the below command
      <br> <i> "modprobe kvm_intel" </i>
      <br> <i> "modprobe kvm" </i>
7. Start the nested VM
8. Tail the dmesg logs using the command <br><i> dmesg -w </i></br>


The screenshot below shows the number of exits for each exit code.
<img width="856" alt="Screenshot1" src="https://user-images.githubusercontent.com/78173506/143724234-a09e4691-563d-4b87-bad1-e89ac54e96d4.png">

<img width="878" alt="Screenshot2" src="https://user-images.githubusercontent.com/78173506/143724245-3fa6f4c1-e178-43fe-be1b-c2afa1e9457c.png">

<img width="836" alt="Screenshot3" src="https://user-images.githubusercontent.com/78173506/143724249-bf44af4f-2223-4f85-a09a-2c28788facab.png">

<img width="854" alt="Screenshot4" src="https://user-images.githubusercontent.com/78173506/143724254-50e9ef39-9e8b-4a25-8c20-103ef190eee9.png">

<img width="857" alt="Screenshot5" src="https://user-images.githubusercontent.com/78173506/143724256-87e4b123-9914-4025-8b88-e50d285b8102.png">

<img width="673" alt="Screenshot6" src="https://user-images.githubusercontent.com/78173506/143724266-d0131f1a-fad0-482b-8283-083cc17ddd7c.png">

<img width="740" alt="Screenshot7" src="https://user-images.githubusercontent.com/78173506/143724267-2c067429-dacc-4a42-a1c5-3f2f9ac36263.png">

<img width="739" alt="Screenshot8" src="https://user-images.githubusercontent.com/78173506/143724270-889e76df-38dc-4236-bed6-1d915fd5f278.png">

<img width="797" alt="Screenshot9" src="https://user-images.githubusercontent.com/78173506/143724285-051acc98-5451-430e-9a41-cedfa7023ac0.png">

<img width="819" alt="Screenshot10" src="https://user-images.githubusercontent.com/78173506/143724294-923d9e5a-ea78-451d-8815-086adaab1e61.png">


The below 2 screenshots denote the <b> rate of increase of exits for exit code = 10 for every 3 secs </b> 
We noticed that the exit counts increased constantly at 74 throughout the 30 sec window.
<img width="622" alt="Rate1" src="https://user-images.githubusercontent.com/78173506/143724300-19a41563-aae9-4ec2-9e3a-02a70ab565bc.png">

<img width="411" alt="Rate2" src="https://user-images.githubusercontent.com/78173506/143724306-5d517a12-d3b9-4d2f-b315-e782a9a4e7fc.png">

For 0x4ffffffd, before rebooting the count is as shown below. <br>
<img width="407" alt="BeforeReboot" src="https://user-images.githubusercontent.com/78173506/143724322-85a82d4d-4543-424d-9271-70b3c8cc7e10.png">

For 0x4ffffffd, after second rebooting the count is as shown below. It is increasing by 50%. <br>
<img width="403" alt="SecondReboot" src="https://user-images.githubusercontent.com/78173506/143724326-0c045628-ef4d-44fb-a1a8-a429a85b0711.png">

For 0x4ffffffd, after third rebooting the count is as shown below. It is increasing by 50%. <br>
<img width="375" alt="ThirdReboot" src="https://user-images.githubusercontent.com/78173506/143724331-cb57d55b-d2e7-4558-8fb8-210f00fc6962.png">

For 0x4ffffffc, the timeshift screenshots are attached below. We have attached only the ones that cause exit.
<img width="459" alt="Timeshift_0" src="https://user-images.githubusercontent.com/78173506/143724341-aacf70c6-d84f-42ee-84c4-0b7cfff81bb1.png">

<img width="519" alt="TimeShift" src="https://user-images.githubusercontent.com/78173506/143724336-edc9f798-ea88-4681-9b20-818d807e823f.png">

Hexadecimal value of number of exits is passed in eax register shown below for 0x4ffffffd for exit code 10. 
<img width="770" alt="Question3_Cli" src="https://user-images.githubusercontent.com/78173506/143724344-6e3a79e9-44af-4e9f-9832-d74e425ebe89.png">

<img width="651" alt="Question4_cli" src="https://user-images.githubusercontent.com/78173506/143724350-02380881-1702-409d-8e26-34bb11666e85.png"><br>

## Question 4

The most frequent exit type is for exit code = 48. <br>
The least frequent exit type is for exit code = 29.


# Assignment 4 - Nested Paging vs. Shadow Paging

<img width="629" alt="Screen Shot 2021-11-27 at 7 17 18 PM" src="https://user-images.githubusercontent.com/78173506/143727535-e9819b7e-4b4a-4dd9-b427-973d5f3101db.png">

<img width="1440" alt="Comparisons1" src="https://user-images.githubusercontent.com/9292835/143727584-b9b57ee7-04fb-4edd-9504-6b87b5dcd9b1.png">

<img width="1440" alt="Comparisons2" src="https://user-images.githubusercontent.com/9292835/143727585-6846fe77-a789-4e66-b178-85f72a579f35.png">



    
