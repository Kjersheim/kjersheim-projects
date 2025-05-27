---
title: "Data Networks Episode 1: Setting up virtual machines"
date: 2023-01-20
draft: false
summary: "Exploring the first steps of network analysis using Lubuntu virtual machines, ping, traceroute, and pathping."
---

# E01 Setting up the first virtual machines

**Part 1:**

Lubuntu - Settings - Networks - Adapter:

MAC-address:\
´080027A3BD08´

Screenshot of the network settings, after setting it to Bridge adapter:\
![](/images/network-doc/E01/Screenshot1NetworkSettings.png)

---

**Part 2:**

Despite not having a connection to the internet, there are still applications and functions on the virtual machine, like an ordinary computer.  
You can have a local connection to your host machine and transfer files, or via an USB-drive.

Alternatively, you can use the VM to test different programs or functions, or in a safe isolated environment learn or test these without doing harm to your host/other files, locations and machines.

Lastly you can crack nazi-codes, or try to test if machines can think. Whichever action that does not require a connection to another computer.

---

**Part 3:**

Using ping to check if I can get a response from www.jamk.fi:

```
lubuntu@lubuntu-virtualbox:~$ ping jamk.fi
PING jamk.fi (94.237.98.172) 56(84) bytes of data.
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=1 ttl=50 time=50.4 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=2 ttl=50 time=37.8 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=3 ttl=50 time=44.9 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=4 ttl=50 time=41.7 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=5 ttl=50 time=51.5 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=6 ttl=50 time=44.3 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=7 ttl=50 time=46.3 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=8 ttl=50 time=48.1 ms
64 bytes from 94-237-98-172.de-fra1.upcloud.host (94.237.98.172): icmp_seq=9 ttl=50 time=48.4 ms
^C
--- jamk.fi ping statistics ---
10 packets transmitted, 9 received, 10% packet loss, time 9004ms
rtt min/avg/max/mdev = 37.845/45.942/51.504/4.066 ms
```

---

**Part 4:**

Traceroute www.jamk.fi

```
lubuntu@lubuntu-virtualbox:~$ traceroute www.jamk.fi
traceroute to www.jamk.fi (94.237.98.172), 64 hops max
  1   192.168.1.1  3,685ms  8,376ms  8,554ms 
  2   *  *  * 
  3   62.78.106.72  26,658ms  22,404ms  21,128ms 
  4   *  *  * 
  5   *  *  * 
  6   *  *  *                                                                                                                                                                                
  7   193.110.224.20  31,360ms  22,348ms  19,525ms                                                                                                                                           
  8   193.110.224.68  18,397ms  23,802ms  28,353ms                                                                                                                                           
  9   94.237.0.112  23,832ms  24,288ms  29,945ms                                                                                                                                             
 10   94.237.0.43  49,109ms  51,279ms  44,926ms                                                                                                                                              
 11   *  *  *                                                                                                                                                                                
 12   *  *  *                                                                                                                                                                                
 13   *  *  *                                                                                                                                                                                
 14   *  *  *                                                                                                                                                                                
 15   *  *  *                                                                                                                                                                                
 16   *  *  *                                                                                                                                                                                
 17   *  *  *                                                                                                                                                                                
 18   *  *  *                                                                                                                                                                                
 19   *  *  *                                                                                                                                                                                
 20   *  *  *                                                                                                                                                                                
 21   *  *  *                                                                                                                                                                                
 22   *  *  *                                                                                                                                                                                
 23   *  *  *                                                                                                                                                                                
 24   * ^C     
```

Traceroute --resolve-hostnames www.jamk.fi

```
lubuntu@lubuntu-virtualbox:~$ traceroute --resolve-hostnames  www.jamk.fi                                                                                                                    
traceroute to www.jamk.fi (94.237.98.172), 64 hops max
  1   192.168.1.1 (_gateway)  4,996ms  6,646ms  8,318ms 
  2   *  *  * 
  3   62.78.106.72 (esp3-er3.dnaip.fi)  18,398ms  31,011ms  28,317ms 
  4   *  *  * 
  5   *  *  * 
  6   *  *  * 
  7   193.110.224.20 (dna.ficix2.ficix.fi)  29,322ms  18,769ms  21,990ms 
  8   193.110.224.68 (upcloud.ficix2.ficix.fi)  25,243ms  26,025ms  25,612ms 
  9   94.237.0.6 (r1-hel1-po1.fi.net.upcloud.com)  26,595ms  22,390ms  25,801ms 
 10   94.237.0.43 (r1-fra1-et3.de.net.upcloud.com)  44,373ms  39,778ms  49,671ms 
 11   94.237.0.43 (r1-fra1-et3.de.net.upcloud.com)  49,248ms  43,388ms  41,715ms 
 12   *  *  * 
 13   *  *  * 
 14   *  *  * 
 15   *  *  * 
 16   *  *  * 
 17   *  *  * 
 18   *  *  * 
 19   * ^C
```

I feel I have somewhat of an understanding of how the networks are connected to each other, though it is curious how much longer traceroute takes than ping. Trying to resolve these questions by Google makes me find that it’s a common question. From the answers I find, the most descriptive would be that the traceroute program sends a series of separate packets to the target, and deliberately tries to get a timeout error, hence it starts with a TTL of 1 — which I found to mean *Time To Live* — and increases this, waiting for a timeout each time.

I want to make a second VM on my laptop to do some testing later on, to double-check connections and the systems and whether these affect the results.

---

**Part 5:**

mtr www.jamk.fi:  
![](/images/network-doc/E01/Screenshot2MtrJamk.png)

mtr -n www.jamk.fi:  
![](/images/network-doc/E01/Screenshot3MtrNJamk.png)

One "intersection" of the route had a massive packet loss, ranging between 50–80%.  
Not really understanding this, but I tried pathping in Windows both on a laptop on the same Wi-Fi — and on the stationary machine that hosts the VM.

**pathping stationary host:**  
![](/images/network-doc/E01/Screenshot4PathpingStationaryHost.png)

**pathping laptop:**  
![](/images/network-doc/E01/Screenshot5PathpingLaptop.jpeg)

From doing similar searches with pathping -n, I find several locations where packet loss is high. From what I understand, use of an asterisk `"*"` means packet loss. I need to go through more of the material and search for an*
