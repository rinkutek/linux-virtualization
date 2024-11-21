
# CMPE283: Assignment 2: KVM Statistics



## Authors

- [Junie Mariam Varghese](https://www.github.com/juniemariam)
- [Rinku Tekchandani](https://github.com/rinkutek)



## Question 1

Rinku took the lead in setting up the kernel. She ensured the base Linux kernel was successfully downloaded, configured, and ready for modifications. She also verified the environment setup, including dependencies and prerequisites, to ensure smooth development and testing.

Junie focused on implementing the required changes in the vmx.c and vmx.h file. This included adding the per-exit-type counters, global exit counters, and the vm_exit_reason_names array for human-readable mapping. She also ensured the kernel was modified correctly and verified the build process.

Both Junie and Rinku worked together to test the modified kernel. Junnie set up and configured the virtualized environment, including the outer and inner VMs, and validated the kernel modifications by monitoring dmesg logs for exit statistics. Rinku supported the testing process by troubleshooting any configuration or environment-related issues.

Rinku handled pushing the modified kernel source code to the GitHub repository, resolving any issues related to repository setup and permissions. She also took the lead in troubleshooting the Git operations and ensuring the code was properly version-controlled and ready for submission. Junie led the documentation process, creating a detailed README file explaining the steps followed, changes made, and results obtained.
## Question 2
## Prerequisites

You will need a working assignment 1 configuration(outer vm and inner vm). We did this assignment in gcp.
```bash
  gcloud compute ssh outer-vm
```
## Step- 1 Setting up the kernel

Clone the project
```bash
  git clone https://github.com/torvalds/linux.git
```

Go to the project directory
```bash
  cd linux
```

Configure the Kernel -> do enable (* or m for 
- Virtualization->Kernel-based Virtual Machine (KVM) support->KVM for Intel processors support
- Device Drivers ->Virtio drivers->PCI driver for virtio devices-> Virtio balloon driver-> Virtio input driver
- Device Drivers ->Network device support ->Universal TUN/TAP device driver support
```bash
  make menuconfig
```

Rebuild the Kernel
```bash
  make -j$(nproc)
```

Build Initrd/Initramfs
```bash
  make modules_install
  make install

```
 Reinstall the Kernel
```bash
  sudo make install
  sudo update-grub
```
Test the Rebuilt Kernel
```bash
  sudo reboot
```
Verify the running kernel:
```bash
  gcloud compute ssh outer-vm
  uname -r
```

## Step- 2  Modifying the kernel
Navigate to the KVM Source Code Directory
```bash
  cd linux
```
Modify the Exit Handler Function files under vmx.c and vmx.h
```bash
  nano arch/x86/kvm/vmx/vmx.c
```
```bash
  nano arch/x86/kvm/vmx/vmx.h
```
Build and Install the Modified Kernel
```bash
  cd linux
  make -j$(nproc)
  sudo make modules_install
  sudo make install
  sudo update-grub
```
Reboot the outer vm
```bash
  sudo reboot
```

## Step- 3  Testing and veryfying the changes in the kernel

Enter the outer vm with modified kernel:
```bash
  gcloud compute ssh outer-vm
```
Verify if the updated KVM and tun module is loaded:
```bash
  lsmod | grep kvm
  lsmod | grep tun
```
To run the inner vm command we do need l2-image.img(Should be present as per the first assignment)
```bash
  wget https://cloud-images.ubuntu.com/minimal/releases/focal/release/ubuntu-20.04-minimal-cloudimg-amd64.img -O l2-image.img
```
Run the inner VM inside the outer VM using your modified KVM 
```bash
  sudo qemu-system-x86_64 -enable-kvm -hda l2-image.img -m 512 -netdev tap,id=mynet0,ifname=tap0,script=no -device virtio-net-pci,netdev=mynet0 -nographic
```
![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Periodically check the outer VM's dmesg logs to see the exit statistics being printed
```bash
  sudo dmesg
```
![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)


## Question 3
Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there
more exits performed during certain VM operations? Approximately how many exits does a full VM
boot entail?

Ans) During a full VM boot, the number of exits increases steadily but not uniformly, with specific operations causing temporary spikes. Exit types like 30 (Unknown) and 28 (Unknown) are the most frequent, accounting for over a million exits, while others like 1 (External Interrupt) and 10 (CPUID) also contribute significantly. These spikes occur during critical VM operations, such as CPU feature initialization or handling external interrupts, reflecting the hypervisor's active role. Overall, a full VM boot entails approximately 2.5 to 3 million exits, with the steady increase punctuated by spikes corresponding to more hypervisor-intensive tasks.


## Question 4
Of the exit types, which are the most frequent? Least?

Ans) Among the exit types, the most frequent is Exit Reason 30 (Unknown), which accounts for over a million exits during the VM's runtime, followed by Exit Reason 28 (Unknown) and Exit Reason 10 (CPUID).The least frequent exit reason is Exit Reason 0 (Exception or NMI).



