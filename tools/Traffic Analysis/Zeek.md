
---
### What is Zeek?

>  **Zeek** (*formerly knows as Bro*) is a ==Network Security Monitoring tool==, useful for traffic analysing. In it's current version it's only available for ==CLI==, although there is an ==alternative tool== called **ZUI** which is the ==GUI== version of **Zeek**. 
>  
   The reason why **Zeek** is different from other ==Intrusion Detection Systems== is that it organises raw network traffic into **unique log files** that indicate exactly the **type of logs** that they contain. It is easier, user and beginner friendly as well as helpful for anyone wanting to ==monitor System Logs==.

---
### How to Install Zeek? 

- In order to install **Zeek** on any type of **Linux Distribution**, the user must complete some specific steps listed bellow:

	1. Go to [Zeek's Website](https://zeek.org/) and *click* on **Get Zeek**.
	2. Spot the current feature release and go to **-> Downloads:** *Linux Binaries*.
	3. On the right side of the window *click* **Binary Packages** and *select* your distribution.
	4. Once you *select* your distribution *copy/paste* the code on your **terminal** and *press* **enter**.
	5. *Press* **enter** on the two pop-up windows and you are ready!

---
### Where to find Zeek?

* After installing **Zeek** you must locate the file where **Zeek** has been installed with its tools. The folder in question is:  `/opt/zeek/bin`
* In order to enter the folder, type in your terminal: `cd /opt/zeek/bin`
* To view the contents of the folder which will be the available tools given by **Zeek** type: `ls`
---
## Zeek vs PCAP files: How to Start

>   To finally start understanding the power of **Zeek** and its features you must find an available ==PCAP (*packet capture*) file== to work with. Click [here](https://www.malware-traffic-analysis.net/) to find a series of available **PCAP files**. Once you download it unzip it in your terminal and you are ready for Zeek!

### Getting Zeek to Start

* Getting **Zeek** to work requires some steps just for it to be able to ==organise the contents== of the **PCAP file** correctly. **First step** is to make some space for it to work on: 
  1. *Enter* the **tmp** folder: `cd /tmp/`  *==The tmp directory stores temporary files for a short period of time==*.  
  2. *Create* a **new folder** under it and *name* it however you like: `mkdir {name of folder}`
  3. *Enter* the **new created folder**: `cd {name of folder}/`


* Now that we have created our folder, the **next step** is to point **Zeek** at the **PCAP file** to start inspecting it: 
    1. *Type* in your **terminal**: `/opt/zeek/bin/zeek -r ~/Downloads/{name of the PCAP}/` . 
       *==The -r means read a pcap file!==*
    2. *Type* `ls` to **sort out** everything inside the **PCAP file** in **seperate** log files.

* Here is a sample of how it should look like: 
![[Pasted image 20260704171523.png]]

---
## Opening Files with Zeek

### Basic Opening:

* To open one of the appearing `.log` files simply type `less -S {name}.log`. 
		`less` ==allows you to view the content one page at a time== 
		`-S` ==flag lets you view everything in a horizontal line and in columns==
* Press `q` on your keyboard to exit the file contents.

### Zeek-cut Tool:

* To open a **specific column header** and view its content you use the tool **zeek-cut** followed by the specified **column header**. Specifically, you must type: `cat {name of log file}.log | /opt/zeek/bin/zeek-cut {name of column header}` . It should look something like this:

	![[Pasted image 20260704172105.png|409]]

* ==To organise it further you can add specific flags to the command:==
	 1. `sort`: to *sort* the file contents
	 2. `uniq -c` : to *display* repeated content only once
	 3. `sort -n`: to *sort* content by numeric count
  The completed command should look like: `cat {name of log file}.log | /opt/zeek/bin/zeek-cut {name of column header} | sort | uniq -c | sort -n`

![[Pasted image 20260704172918.png]]

---
## Detecting Malicious Activity 

 To identify malicious activity **Zeek** helps by creating a file called `pe.log` . **PE or Portable Executable** files are files that contain ==executables or object code==, indicating sometimes even **malicious activity**.

* **One quick way** to identify malicious activity using **Zeek** is by opening a `pe.log` file in search of a **user id**.
		1. *Open* the `pe.log` file : `less -S pe.log`
		2. *Find* the **User ID**, usually its indicated by a *column header*. 
		3. *Copy* the User ID and *use* the command `grep {user id} *. log`
	
	Now you can spot all the activity in every log file within the **PCAP** pointing to the specified **User ID**.
	
* ==Here is an example of what it should look like==: 
   ![[Pasted image 20260704181426.png]]

---

   