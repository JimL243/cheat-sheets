I had encountered the following issue when attempting to migrate a live VM to another host w/in my lab cluster.  
The error received was: 

Currently connected network interface” ‘Network adapter 1’ cannot use network ‘VM Network’, because “the destination network on the destination host is configured for different offload or security policies than the source network on the source host”.

I was able to fix this by checking the configuration of the virtual switch (vSwitch0) on the ESXi host I was moving the virtual machine guest to.

1. I click on each host went to the configure
2. Under the Networking subsection located the virtual switch
3. Selected edit on that virtual switch.  
    ![](http://i.imgur.com/aeS83YO.png)
4. Reviewed the settings in the Security tab and the Traffic Shaping tab between the hosts.

In my case the issue was with the Security tab.  The destination host did not match the source.  
Just another reasons to use [host profiles](http://jermsmit.com/vmware-vcenter-66-5-creating-host-profiles/) between systems so that settings all match.

Reference: https://jermsmit.com/solved-unable-to-migrate-vms-to-other-host/