---
layout: post
title: "FlagYard Compressed Confession Forensics Challenge"
categories: [CTF Writeup, Forensics]
tags: [FlagYard Writeup, Compressed Confession Challenge, Forensics]
description: "Step-by-Step Writeup for FlagYard's Compressed Confession Forensics Challenge"
---

# Challenge Overview
This writeup provides a step-by-step solution to the **"Compressed Confession"** challenge, which is worth **120 points** in the Forensics category on the FlagYard platform.

![Challenge Description](https://i.ibb.co/hJydzVQW/image.png)

The challenge is a **Windows Registry Hive Forensics** task that involves analyzing Windows Registry data files (hives) to uncover user activities, system configurations, and potential security artifacts.  

While there are many tools available for tackling Windows registry forensics, in this challenge I’ll be using **[Registry Explorer](https://www.sans.org/tools/registry-explorer)** by Eric Zimmerman.
# Step-by-Step Solution
In the attachments, you are given a zipped Windows Registry Hive.  
Unzip it and go to:

`C:\Users\FlagYard\`

Inside this folder, you will see these files:
- NTUSER.DAT (the main user registry hive)  
- ntuser.dat.LOG1 (log file)  
- ntuser.dat.LOG2 (log file)
## Loading the Hive
1. Download and install Registry Explorer from [here](https://www.sans.org/tools/registry-explorer).  
2. Open NTUSER.DAT in Registry Explorer. 
3. A popup will ask if you want to load the log files. Hold Ctrl and select both ntuser.dat.LOG1 and ntuser.dat.LOG2, then press OK.  
4. Save this updated hive as NTUSER.DAT_clean.  
5. Close and re-open NTUSER.DAT_clean in Registry Explorer.

![Hive Loaded](https://i.ibb.co/DDRm9w06/Screenshot-2025-08-31-145815.jpg)

## Finding the Flag
1. In NTUSER.DAT_clean, expand the Root directory.  
2. Navigate to:  
   `Root > Software`
3. Only a few software entries are listed. This draws our focus towards 7-Zip and Microsoft.
4. Expand 7-Zip, then select Compression.  
5. On the right-hand panel, check the Value Stacks.  
6. Open Arc History.

![Encoded Value String](https://i.ibb.co/6JspxQ3t/image-1.jpg)

You will find a base64-encoded string here.
## Decoding the String
The string ends with `=`, which is a common indicator of base64 encoding.  
Open [CyberChef](https://cyberchef.io), paste the string, and apply **Base64 Decode**. 

![Base64 Decoded](https://i.ibb.co/2zHTBY1/image.png)

The decoded output reveals that it has been XOR-encrypted with the key `01`.  
Apply **XOR Decrypt** using the key `01`, and the flag will be revealed.

![XOR Decrypted](https://i.ibb.co/nMKZ4ZDq/image.png)

**Flag:** `FlagY{c002ae7b19e980cf07debb55c8d57450}`

---

Happy Hunting! Follow me on [Twitter/X](https://twitter.com/an0nbil) or connect to me on [LinkedIn](https://www.linkedin.com/in/realbilalsafdar/), you can also DM me on [Discord](https://discordapp.com/users/1275773488354824253) for any queries!
