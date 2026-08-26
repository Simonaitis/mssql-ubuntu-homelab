# Microsoft SQL Server 2022 on Ubuntu 22.04 LTS — Lab Log

## Project Overview
Deployment, network configuration, dependency resolution, and database restoration for MS SQL Server 2022 running on an Ubuntu Server host (`10.0.0.179`), managed remotely via SQL Server Management Studio (SSMS 20).

## Infrastructure Details
* **Host Operating System:** Ubuntu Server 22.04 LTS
* **Database Engine:** Microsoft SQL Server 2022
* **Management Client:** SQL Server Management Studio (SSMS 20) on Windows
* **Sample Dataset:** `AdventureWorks2022`

---

## Deployment & Troubleshooting History

### 1. Host Name & Package Resolution Failure (DNS / Netplan)
* **Problem:** `wget` and `apt` operations failed with `unable to resolve host address`.
* **Root Cause:** Active Netplan configuration lacked external public DNS forwarders.
* **Resolution:** Injected Google Public DNS (`8.8.8.8`) into `/etc/resolv.conf` and updated Netplan configuration.

### 2. Missing Engine Dependency (`libldap-2.5-0`)
* **Problem:** SQL Server installation failed due to a missing shared library (`libldap-2.5.so.0`), which is deprecated/removed in Ubuntu 22.04 default repos.
* **Resolution:** Manually pulled the official `libldap-2.5-0_2.5.16` `.deb` package from Ubuntu security mirrors using `wget` and installed it via `dpkg -i`.

### 3. Service Constraint on `sa` Credential Reset
* **Problem:** Attempting to run `sudo /opt/mssql/bin/mssql-conf set-sa-password` threw an error stating an instance of SQL Server was actively running.
* **Resolution:** Stopped the database service using `systemctl`, executed the reset configuration utility, and restarted the engine:
  ```bash
  sudo systemctl stop mssql-server
  sudo /opt/mssql/bin/mssql-conf set-sa-password
  sudo systemctl start mssql-server

### 4. Database Restore Path & File Ownership Fixes
**Troubleshooting: AdventureWorks Restore Failure**

* **Error:** `Msg 3201, Level 16 - Operating System Error 2 (The system cannot find the file specified)`
* **Root Cause:** Target backup filename contained a typo (`AdeventureWorks2022.bak`), and the `mssql` service account lacked file ownership.
* **Fix Applied:**
  ```bash
  # Correct filename typo
  sudo mv /var/opt/mssql/data/AdeventureWorks2022.bak /var/opt/mssql/data/AdventureWorks2022.bak

  # Set mssql service account ownership
  sudo chown mssql:mssql /var/opt/mssql/data/AdventureWorks2022.bak