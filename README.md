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
