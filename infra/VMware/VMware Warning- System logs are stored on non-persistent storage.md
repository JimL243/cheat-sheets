
If you have ever installed VMware’s ESXi OS on a USB storage drive or SD card, you may have seen the following warning: System logs are stored on non-persistent storage

![](https://www.labcrasher.com/wp-content/uploads/2021/08/Error-1024x346.jpg)

Warning: System logs are stored on non-persistent storage

In order to rectify this issue, you simply need to change the global syslog location in the advanced system settings and the warning will disappear.  The steps below outline how to do this from the vCenter vSphere HTML5 interface, though similar steps are taken from the ESXi web UI.

1. On the host with the warning, go to the Datastores tab and select a datastore for the log files
2. Right-click that datastore and select Browse Files  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-2.jpg)
3. Select New Folder in the datastore browser  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-3.jpg)
4. Name the folder something recognizable, like “logs”.  Then click OK and confirm the folder shows in the datastore browser  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-4.jpg)
5. Head over to the Configure tab on that host, then go to Advanced System Settings and click Edit  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-5.jpg)
6. On the Edit Advanced System Settings window, filter for **Syslog.global.logDir** and you should be presented with the settings in the screenshot  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-6.jpg)
7. Modify the value of Syslog.global.logDir with this format: [**DatastoreName**] /**logFolder**.  The log directory I chose is [ESXI01-VMDS01] /logs.  Make sure to include a space between the “]” and “/”.  Once updated, click OK  
    ![](https://labcrasher.com/wp-content/uploads/2021/08/SystemLogs-Step-7.jpg)

When you return to the Summary tab, you’ll see that the System Logs warning on this host is no longer present!  No rinse and repeat for any host that is still showing this warning.  If you are using shared storage, such as a SAN, NAS, or dHCI solution, this method still applies.  My only suggestion would be to create a log directory for each host so that multiple hosts don’t use the same directory.

![](https://www.labcrasher.com/wp-content/uploads/2021/08/Result-1024x325.jpg)

The warning for system logs has cleared

The link to the official VMware KB article for this warning is here: [https://kb.vmware.com/s/article/2032823](https://kb.vmware.com/s/article/2032823)

If you also see the warnings in the screenshot above, you should read [how to disable SSH warnings](https://www.labcrasher.com/2021/08/18/vmware-warning-ssh-or-esxi-shell-for-the-host-has-been-enabled/) and [fixing coredump target warnings](https://www.labcrasher.com/2021/08/18/vmware-warning-no-coredump-target-has-been-configured/)!

Reference: https://labcrasher.com/2021/08/18/vmware-warning-system-logs-are-stored-on-non-persistent-storage/