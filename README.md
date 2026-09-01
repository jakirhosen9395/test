# Device Information Collection Guide

Use the instructions below to collect the required device information for the company asset inventory.

## Required Information

For each employee, collect:

| Field                           | Required Information                      |
| ------------------------------- | ----------------------------------------- |
| **Employee Name**               | Employee's full name                      |
| **Device HostName**             | Company-assigned hostname                 |
| **WiFi-MAC Address**            | MAC address of the Wi-Fi adapter          |
| **LAN-MAC Address**             | MAC address of the Ethernet/LAN adapter   |
| **Device Model, CPU, RAM, HDD** | Manufacturer/Model, CPU, RAM, and Storage |

> **Important:** If a device does not have Wi-Fi or Ethernet, enter `N/A` for that field.

---

# Windows

## 1. Set Device Hostname

Run **PowerShell as Administrator**:

```powershell
Rename-Computer -NewName "Replace-With-Your-Device-HostName"
```

### Example

```powershell
Rename-Computer -NewName "GIX-IT-001-Jakir"
```

### Example Output

```text
WARNING: The changes will take effect after you restart the computer.
```

Restart the computer after running the command.

---

## 2. Verify Device Hostname

Run:

```powershell
hostname
```

### Example Output

```text
GIX-IT-001-Jakir
```

Record this value under:

**Device HostName**

---

## 3. Get Wi-Fi and LAN MAC Addresses

Run:

```powershell
Get-NetAdapter | Select-Object Name, InterfaceDescription, MacAddress, Status
```

### Example Output

```text
Name       InterfaceDescription                    MacAddress        Status
----       --------------------                    ----------        ------
Ethernet   Realtek PCIe GbE Family Controller     3C-52-82-AB-12-34 Up
Wi-Fi      Intel(R) Wi-Fi 6 AX201 160MHz          A8-41-F4-CD-56-78 Up
```

Identify the MAC addresses as follows:

```text
Wi-Fi MAC:
A8-41-F4-CD-56-78

LAN MAC:
3C-52-82-AB-12-34
```

Record them under:

* **WiFi-MAC Address**
* **LAN-MAC Address**

> If Ethernet is not connected, the adapter may show `Disconnected`. The MAC address can still be recorded.

---

## 4. Get Device Model, CPU, RAM and Storage

Run **PowerShell**:

```powershell
$pc=Get-CimInstance Win32_ComputerSystem; $cpu=Get-CimInstance Win32_Processor | Select-Object -First 1; $disk=Get-PhysicalDisk; [PSCustomObject]@{Manufacturer=$pc.Manufacturer; Model=$pc.Model; CPU=$cpu.Name; "CPU Cores"=$cpu.NumberOfCores; "Logical Processors"=$cpu.NumberOfLogicalProcessors; "RAM(GB)"=[math]::Round($pc.TotalPhysicalMemory/1GB,2); "Storage(GB)"=([math]::Round(($disk | Measure-Object Size -Sum).Sum/1GB,2))}
```

### Example Output

```text
Manufacturer          : Dell Inc.
Model                 : Latitude 5420
CPU                   : 11th Gen Intel(R) Core(TM) i5-1145G7 @ 2.60GHz
CPU Cores             : 4
Logical Processors    : 8
RAM(GB)               : 16
Storage(GB)           : 476.94
```

### Record as

```text
Dell Latitude 5420
Intel Core i5-1145G7
16 GB RAM
512 GB SSD
```

> For the inventory, you can round storage to the manufacturer's advertised capacity, e.g. `476.94 GB` → `512 GB SSD`.

---

# macOS

## 1. Set Device Hostname

Open **Terminal** and run:

```bash
sudo scutil --set HostName "Replace-With-Your-Device-HostName"
```

### Example

```bash
sudo scutil --set HostName "GIX-IT-001-Jakir"
```

Enter the Mac administrator password when prompted.

### Verify Hostname

Run:

```bash
scutil --get HostName
```

### Example Output

```text
GIX-IT-001-Jakir
```

Record this value under:

**Device HostName**

---

## 2. Get Wi-Fi and LAN MAC Addresses

Run:

```bash
networksetup -listallhardwareports
```

### Example Output

```text
Hardware Port: Wi-Fi
Device: en0
Ethernet Address: a8:41:f4:cd:56:78

Hardware Port: USB 10/100/1000 LAN
Device: en5
Ethernet Address: 3c:52:82:ab:12:34

Hardware Port: Bluetooth PAN
Device: en6
Ethernet Address: 00:11:22:33:44:55
```

### Record

```text
Wi-Fi MAC:
a8:41:f4:cd:56:78

LAN MAC:
3c:52:82:ab:12:34
```

> **Do not record the Bluetooth MAC address.**

> On some MacBooks, Ethernet is provided through a USB-C/Thunderbolt adapter or dock. In that case, record the MAC address belonging to the Ethernet/LAN adapter.

---

## 3. Get Device Model, CPU, RAM and Storage

Run:

```bash
system_profiler SPHardwareDataType SPNVMeDataType
```

### Example Output

```text
Hardware:

    Hardware Overview:

      Model Name: MacBook Pro
      Model Identifier: MacBookPro18,3
      Chip: Apple M1 Pro
      Total Number of Cores: 10
      Memory: 16 GB
      System Firmware Version: 1234.0.0.0.0
      Serial Number (system): XXXXXXXX

NVMe:

    APPLE SSD AP1024R:
      Capacity: 1 TB
```

### Record as

```text
MacBook Pro
Apple M1 Pro
16 GB RAM
1 TB SSD
```

> **Do not include the Mac serial number in the shared inventory unless it is specifically required.**

---

# Linux

## 1. Set Device Hostname

Open **Terminal** and run:

```bash
sudo hostnamectl set-hostname "Replace-With-Your-Device-HostName"
```

### Example

```bash
sudo hostnamectl set-hostname "GIX-IT-001-Jakir"
```

A restart is normally not required.

---

## 2. Verify Hostname

Run:

```bash
hostname
```

### Example Output

```text
GIX-IT-001-Jakir
```

Record this value under:

**Device HostName**

---

## 3. Get Wi-Fi and LAN MAC Addresses

Run:

```bash
nmcli -f DEVICE,TYPE,STATE,CONNECTION device status
```

### Example Output

```text
DEVICE   TYPE      STATE      CONNECTION
enp3s0   ethernet  connected  Wired connection 1
wlp2s0   wifi      connected  Office-WiFi
lo       loopback  unmanaged  --
```

This identifies the network interfaces.

Now run:

```bash
nmcli -f GENERAL.DEVICE,GENERAL.TYPE,GENERAL.HWADDR device show
```

### Example Output

```text
GENERAL.DEVICE:                     enp3s0
GENERAL.TYPE:                       ethernet
GENERAL.HWADDR:                     3C:52:82:AB:12:34

GENERAL.DEVICE:                     wlp2s0
GENERAL.TYPE:                       wifi
GENERAL.HWADDR:                     A8:41:F4:CD:56:78

GENERAL.DEVICE:                     lo
GENERAL.TYPE:                       loopback
GENERAL.HWADDR:                     --
```

### Record

```text
Wi-Fi MAC:
A8:41:F4:CD:56:78

LAN MAC:
3C:52:82:AB:12:34
```

> **Do not record the loopback interface (`lo`).**

---

## 4. Get Device Model, CPU, RAM and Storage

Run:

```bash
echo "Manufacturer: $(sudo dmidecode -s system-manufacturer)"
echo "Model: $(sudo dmidecode -s system-product-name)"
echo "CPU: $(lscpu | grep 'Model name' | sed 's/.*: *//')"
echo "CPU Cores: $(lscpu | awk '/^Core\(s\) per socket:/ {print $4}')"
echo "Logical Processors: $(nproc)"
echo "RAM: $(free -h | awk '/Mem:/ {print $2}')"
echo "Storage:"
lsblk -d -o NAME,SIZE,MODEL | tail -n +2
```

### Example Output

```text
Manufacturer: Dell Inc.
Model: Latitude 5420
CPU: 11th Gen Intel(R) Core(TM) i5-1145G7 @ 2.60GHz
CPU Cores: 4
Logical Processors: 8
RAM: 15Gi

Storage:
NAME  SIZE MODEL
sda   477G KBG40ZNV512G KIOXIA
```

### Record as

```text
Dell Latitude 5420
Intel Core i5-1145G7
16 GB RAM
512 GB SSD
```

---


# Important Notes

## MAC Address

Use the **hardware MAC address**, not an IP address.

Example:

```text
Wi-Fi MAC: A8-41-F4-CD-56-78
LAN MAC:   3C-52-82-AB-12-34
```

Do not use:

```text
192.168.1.10
10.10.30.25
```

These are IP addresses, not MAC addresses.

---

## If Wi-Fi or LAN is unavailable

Use:

```text
N/A
```

For example, a MacBook without an Ethernet adapter:

| WiFi-MAC Address  | LAN-MAC Address |
| ----------------- | --------------- |
| A8:41:F4:CD:56:78 | N/A             |

---

## Storage Naming

For consistency, record storage as:

```text
512 GB SSD
1 TB SSD
1 TB HDD
2 TB HDD
```

rather than copying only the operating system's exact usable capacity.

---

## Recommended Final Format

For the **Device Model, CPU, RAM, HDD** column, use:

```text
[Manufacturer] [Model], [CPU], [RAM], [Storage]
```

Example:

```text
Dell Latitude 5420, Intel Core i5-1145G7, 16 GB RAM, 512 GB SSD
```

This keeps the entire employee device inventory consistent across **Windows, macOS, and Linux**.
