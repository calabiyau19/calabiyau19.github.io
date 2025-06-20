---
layout: post
title: "Attempt 2 to fix Epson ET4850 printer on home network"
draft: false88
date:  2025-04-23
description: Continued problems with losing the Epson ET-4850 printer connection over the network led me to a series of troubleshooting steps to resolve it today.  
---
**06-20-25 UPDATE**: Since 5-18, the printer has been working fine and not lost connection one time. The ping test was stopped, the 192.168.1.92 IP address was checked to make sure it was a fixed or static IP on the network. But, the real fix was changing the printer's network to 2Wire instead of Deco. The working theory is with three Deco access points and 2.5 and 5 options on each, the printer was roaming between dco connections and losing access from time to time that only a hard reboot seemed to fix. Printers don't roam well like cellphones, tablets and laptops as well. I also isolated the deco access to the nearest one and disabled band steering but forgot how I did it.  Probably in Deco configuration. 

In the menu, in settings, reset default network settings, then immediately reboot the printer, hit the Wi-Fi icon, and the Wi-Fi setup wizard started and I reconnected it to my 2wire network, not my Deco mesh network, and the printer was back online again. Also stopped the pinging script because it was doing nothing and potentially a cause for loss of network connectivity. 

The final step was deleting the printer Chad set up in CUPS and now I am back to only having one working printer. 
Also got into the Epson web interface at http://192.168.1.92 and made sure obtain IP was set to manual and matched what was in the router. Static. 
Then as I needed a better tool to monitor up/down status - I installed LibreNMS which then led to every device now running on LibreNMS. So, constant monitoring via SMTP which seems to be working. 


**05-18-25 UPDATE**:  I should not have said anything.  The next day it was a major problem again not solved with a simple re-boot of the printer.

**05-15-25 UPDATE**:  7 weeks in and the printer has never disconnected and is working without error

The **5-18-25 UPDATE**: fix was to reset network settings in Settings, immediately reboot the printer and do the wifi wizard again.  Using the long cat 5 cable did allow me to print but did not solve the wifi issue this time.  In the process I deleted the CUPS installed printer Chad had me set up and everything prints to the printer fine now.  Have no idea why it keeps happening other than it is a printer issue as it is all over online help forums with these Epson printers.



### **1\. Open the CUPS Web Interface**

**Command:**

```sh
sudo xdg-open http://localhost:631
```

Or go to [http://localhost:631](http://localhost:631) to access the CUPS administration interface

**Explanation:** Opens the web UI for CUPS, where you can manage printers directly.

---

### **2\. Delete the Broken Printer**

**Action:**

 In CUPS GUI: `Printers > [Broken Printer] > Delete Printer`

**Explanation:** Removes any misconfigured or non-functional printer entries

I was unable to delete the printer in the GUI interface, so I ran the following command. 

```sh 
sudo nano /etc/cups/printers.conf

```

This opened printers.conf, where I saw my two printers listed. One was real, one was a phantom. Deleted the phantom. Then went to the next step. 

---

### **3\. Re-Add the Printer via CUPS**

**Steps:**

* Go to `Administration > Add Printer`

* Select your Epson printer (preferably IPP/driverless)

* Use **"IPP Everywhere"** if available.

**Explanation:** Adds a clean printer config, avoiding old SSL/PKI issues.

---

### **4\. Restart CUPS**

**Command:**

```sh
sudo systemctl restart cups

```

**Explanation:** Applies config changes and resets the print service.

---

### **5\. Confirm Working Printer & Cleanup**

* Tried printing from both printer entries

* Set both to match the working config.

* Verified printing succeeded

**Explanation:** Ensured settings were consistent, and both entries pointed to the same functional backend.

---

### **6\. Deletion of duplicate printers**

At this point, I realized we were having trouble deleting the printer that was not working correctly. So, we went in and performed some additional manual commands. 

```sh
sudo nano /etc/cups/printers.conf

```

Delete nonworking `<Printer ...>` blocks. Then restart CUPS:

```sh  
sudo systemctl restart cups

```

At this point, we determined that **cups-browsed** was running as normal and locating old printer configuration files on other devices, and restoring them to the printer listing on the Linux box. We needed to stop this so:	

```sh 
sudo systemctl disable \--now cups-browsed

```

### **7\. Set the working printer as the default**

```sh 
lpoptions -d EPSON_ET-4850_Series  
```

Confirmed it with:

```sh 
lpstat -p -d  
```

Only the correct printer remained and was set as the default.

---

### **Optional Backup**

You saved your clean config:

```sh  
sudo cp /etc/cups/printers.conf ~/printers.conf.backup  
```

---

## **Final Working State**

* One printer entry: `EPSON_ET-4850_Series`

* No more phantom printers

* `cups-browsed` disabled permanently

* Prints successfully from Linux Mint, Windows, and iOS

* The issue no longer recurs on reboot or service restart.

