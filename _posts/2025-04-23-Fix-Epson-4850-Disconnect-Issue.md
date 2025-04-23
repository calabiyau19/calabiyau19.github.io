---
layout: post
title: "Attempt 1 to fix Epson ET4850 printer on home network"
draft: false
date:  2025-04-23
description: Continued problems with losing the Epson ET-4850 printer connection over the network led me to a series of troubleshooting steps to resolve it today.  
---


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

