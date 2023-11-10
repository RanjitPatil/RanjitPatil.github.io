---
title: "BiBi Windows Wiper Analysis"
classes: wide
tags: [Wiper Malware]
header:
    teaser: /assets/images/bibi.png
description: "On 30th October, Security Joes posted its findings about a Wiper malware for Linux systems used in the war in Gaza. It was called the "BiBi-Linux Wiper". And then on November 1 2023, BlackBerry Research and Intelligence Team found its Windows variant."

categories:
  - Malware Analysis
toc: true
ribbon: red
---

# BiBi-Windows-Wiper-Analysis

On 30th October, Security Joes posted its findings about a Wiper malware for Linux systems used in the war in Gaza. It was called the "BiBi-Linux Wiper". And then on November 1 2023, BlackBerry Research and Intelligence Team found its Windows variant.

In this post, we will look at the Windows version of the BiBi Wiper known as the "BiBi-Windows Wiper"

## Malware Sample

> **MD5:** e26bba0304f14ef96beb60376791d32c

> **SHA256:** 40417e937cd244b2f928150cae6fa0eff5551fdb401ea072f6ecdda67a747e17

## Static Analysis

- The timestamp suggests the implant was compiled on Saturday, October 21, 2023, and it's a 64-bit one.

  ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/9251f133-59dc-41e5-bfef-c186bdbbe429)

- Below are some intresting strings found in the binary file.

  ```
  [+] Stats: %d | %d
  [!] Waiting For Queue
  [+] Round %d
  lla/ teIuq/ swodahs   eteled nimdassv  c/ exe.dmc
  eteled ypocwodahs cimw c/ exe.dmc
  seruliafllaerongi ycilopsutatstoob tluafed{ tes / tidedcb c / exe.dmc
  on delbaneyrevocer }tluafed{ tes/ tidedcb c/ exe.dmc
  C:\Users
  [+] Path: %s
  [+] CPU cores: %d, Threads: %d
  .exe
  .dll
  .sys
  .BiBi
  
  ```
  
## Dynamic Analysis 

- Upon execution, the BiBi-Windows Wiper checks to see if any arguments have been passed to the BiBi Wiper to destroy the directory, If no argument is provided then it performs the following actions.

- Wiper fetches the number of processors, calculates the threads accordingly using ```GetNativeSystemInfo()``` and prints the target directories and thread information on the console.

  ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/6e861f06-9bfa-484b-8e33-9fe210bf5080)

- Then it reads the hardcoded path: "C:\Users".

  ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/2ffc724a-5589-4908-aba7-97d85b3947d8)

- After that it Iterates through the A-Z (26) disk drives using ```GetLogicalDrives()```, where the return result is the bitmask. It next does a bittest with the received bitmask to determine the system's accessible drives and appends ":" to the drive name.

- Then, except for the C drive, it calls the ```GetDriveTypeA()``` function, which returns the drive type. The BiBi-Windows Wiper exclusively targets the following drive types:

    DRIVE_FIXED

    DRIVE_REMOVABLE

    DRIVE_RAMDISK

  ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/17c36a31-5357-47c1-9808-d81c5152d290)

- Further, it creates a new thread that reads the commands stored in reverse and then creates a new process using ```CreateProcessA()``` to execute those commands.

  ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/58b8cf73-4814-4d49-8f21-4e5cc5281e61)

- Following are the commands executed by Bibi Wiper.

    1.  `cmd.exe /c bcdedit /set {default} recoveryenabled no` - Disables Windows Recovery Environment
    
    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/e46067ef-fec9-49fb-9f9d-501d8f831e30)
    
    2.  `cmd.exe / c bcdedit / set {default} bootstatuspolicy ignoreallfailures` - Force the system to boot normally rather than into the Windows Recovery Environment

    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/4703d6f2-c279-42d3-99a5-82e93058c994)
    
    3.  `cmd.exe /c wmic shadowcopy delete`  - Delete Volume Shadow Copies using WMIC
    
    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/dd7ff501-a1b8-4798-992a-a2e9bc1afb4e)

    4.  `cmd.exe /c vssadmin delete shadows /quIet /all` - Delete Volume Shadow Copies using VssAdmin

    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/ff89dacb-12ef-47fd-9086-21d26c15256b)

 -  After identifying accessible drives and their types, BiBi-Windows Wiper takes additional steps by creating a separate thread to execute the main wiping routines. These routines require two arguments: the path of the directory to be destroyed (either provided by the operator or retrieved earlier) and the specified number of threads.

 -  The wiper then enters an infinite loop where the counter corresponds to the round number, printing "[+] Round %d\n" for each iteration. This indicates that once initiated, the Wiper continuously destroys data in an infinite loop.

 -  To optimize the process, the wiper creates multiple threads based on the specified number, executing the main wiping function within a loop. Notably, the BiBi-Windows Wiper is designed to exclude files with ".exe," ".dll," and ".sys" extensions from its destructive actions.

    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/fa9c594d-d9d0-4329-a369-7b7877436fd0)

-  The Wiper function implements the Mersenne Twister PseudoRandom Number Generator Algorithm which generates random numbers. Then Wiper changes the name of the destroyed files using the Mersenne Twister function again. The generated random number undergoes a modulus operation with a hardcoded value, creating an index in a wide string. This index is then used to form a unique filename, appended with ".BiBi" and the round number.

    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/b5ad5de1-60eb-4261-9448-344a62fc8e17)

   ```
   v2 = 624i64;
   v3 = *a1;
    if ( v3 == 624 )
    {
    v4 = a1 + 2;
    do
    {
      v5 = *v4 ^ *(v4 - 1);
      ++v4;
      v4[622] = ((*(v4 - 2) ^ v5 & 0x7FFFFFFFu) >> 1) ^ v4[395] ^ (((*((_BYTE *)v4 - 8) ^ (unsigned __int8)v5) & 1) != 0 ? 0x9908B0DF : 0);
      --v2;
    }
    while ( v2 );
    v3 = *a1;
    }
    else if ( v3 >= 0x4E0 )
    {
    v6 = a1 + 625;
    v7 = a1[625];
    v8 = 227i64;
    do
    {
      v9 = v7 ^ (v7 ^ v6[1]) & 0x7FFFFFFF;
      v7 = v6[1];
      *(v6 - 624) = (v9 >> 1) ^ v6[397] ^ ((v6[1] & 1) != 0 ? 0x9908B0DF : 0);
      ++v6;
      --v8;
    }
    while ( v8 );
    v10 = a1 + 852;
    v11 = 396i64;
    v12 = a1[852];
    do
    {
      v13 = v12 ^ (v10[1] ^ v12) & 0x7FFFFFFF;
      v12 = v10[1];
      *(v10 - 624) = (v13 >> 1) ^ *(v10 - 851) ^ ((v10[1] & 1) != 0 ? 0x9908B0DF : 0);
      ++v10;
      --v11;
    }
    while ( v11 );
    a1[624] = ((a1[1248] ^ (a1[1] ^ a1[1248]) & 0x7FFFFFFF) >> 1) ^ a1[397] ^ ((a1[1] & 1) != 0 ? 0x9908B0DF : 0);
    v3 = 0;
    *a1 = 0;
    }
    v14 = a1[v3 + 1];
    *a1 = v3 + 1;
    v15 = ((((v14 >> 11) & a1[1249] ^ v14) & 0xFF3A58AD) << 7) ^ (v14 >> 11) & a1[1249] ^ v14;
    return ((v15 & 0xFFFFDF8C) << 15) ^ v15 ^ ((((v15 & 0xFFFFDF8C) << 15) ^ v15) >> 18);
    }
   
    ```
-  Below output shows the Target directory, CPU Cores, Threads, Round Number, Stats, and destroyed file with .BiBi extension.

    ![image](https://github.com/RanjitPatil/BiBi-Wiper/assets/43460691/d8e78456-66a9-47fd-a73e-ada5faf7bb9a)
   

## YARA Rule

```
rule BIBI_Wiper_Windows {

meta:

    description ="BiBi-Windows Wiper used in the Gaza War"
    author ="The BlackBerry Research and Intelligence Team"
    date = "2023-10-31"
    hash ="40417e937cd244b2f928150cae6fa0eff5551fdb401ea072f6ecdda67a747e17"
    version = "1.0"

strings:
    
    $a1 = "[+] Stats: " ascii wide 
    $a2 = "C:\\Users" ascii wide 
    $a3 = "[!] Waiting For Queue " ascii wide
    $a4 = "[+] Round " ascii wide
    $a5 = "[+] Path: " ascii wide
    $a6 = "[+] CPU cores: " ascii wide

condition:
    uint16(0) == 0x5a4d and ((filesize < 2000KB) and all of ($a*))
}

```

## References :

- [https://www.linkedin.com/pulse/bibi-wiper-gaza-war-now-goes-windows-dmitry-bestuzhev-yftze/#](https://www.linkedin.com/pulse/bibi-wiper-gaza-war-now-goes-windows-dmitry-bestuzhev-yftze/#)