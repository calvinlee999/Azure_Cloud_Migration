# Server Discovery Template

## Instructions
1. Download this CSV template
2. Fill in your server inventory data
3. Import to Azure Migrate for quick assessment

## CSV Format

```csv
Machine name,IP address,Operating system,Cores,Memory (MB),Disk 1 (GB),Disk 2 (GB),Disk 3 (GB),Boot type
```

## Example Data

```csv
Machine name,IP address,Operating system,Cores,Memory (MB),Disk 1 (GB),Disk 2 (GB),Disk 3 (GB),Boot type
web-server-01,10.0.1.10,Windows Server 2019,4,16384,128,500,,UEFI
db-server-01,10.0.1.20,Windows Server 2019,8,32768,128,1000,,UEFI
app-server-01,10.0.1.30,Ubuntu 20.04,4,16384,64,250,,UEFI
web-server-02,10.0.1.11,Windows Server 2019,4,16384,128,500,,UEFI
cache-server-01,10.0.1.40,Ubuntu 20.04,2,8192,64,,,UEFI
```

## Field Descriptions

| Field | Description | Required | Notes |
|-------|-------------|----------|-------|
| Machine name | Server hostname | Yes | Must be unique |
| IP address | IPv4 address | Yes | Internal IP preferred |
| Operating system | OS name and version | Yes | Be specific (include version) |
| Cores | Number of CPU cores | Yes | Physical or vCPU count |
| Memory (MB) | RAM in megabytes | Yes | 1 GB = 1024 MB |
| Disk 1-3 (GB) | Disk sizes in GB | Disk 1 required | Leave blank if not applicable |
| Boot type | BIOS or UEFI | Yes | Check VM settings |

## Tips for Data Collection

### Windows Servers
```powershell
# Get system information
Get-ComputerInfo | Select-Object CsName, WindowsVersion, CsProcessors, CsTotalPhysicalMemory

# Get disk information
Get-Disk | Select-Object Number, Size, FriendlyName

# Check boot type
bcdedit | findstr /i "path"
```

### Linux Servers
```bash
# Get hostname
hostname

# Get OS information
cat /etc/os-release

# Get CPU cores
nproc

# Get memory in MB
free -m | grep Mem | awk '{print $2}'

# Get disk sizes
lsblk -o NAME,SIZE,TYPE | grep disk

# Check boot type (UEFI if directory exists)
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"
```

## VMware RVTools Export

If using VMware, you can use RVTools to generate inventory:

1. Download [RVTools](https://www.robware.net/rvtools/)
2. Connect to vCenter Server
3. Go to vInfo tab
4. Click Export > Export all to Excel
5. Use the resulting XLSX file with Azure Migrate

## Application Dependency Information

To capture application dependencies:

| Field | Description |
|-------|-------------|
| Server Name | Name of the server |
| Application | Application name running on server |
| Communicates With | Other servers this server communicates with |
| Port/Protocol | Ports and protocols used |
| Criticality | Business criticality (Low/Medium/High/Critical) |

Example:
```csv
Server Name,Application,Communicates With,Port/Protocol,Criticality
web-server-01,IIS Web Server,app-server-01,8080/TCP,High
app-server-01,.NET App,db-server-01,1433/TCP,Critical
db-server-01,SQL Server,cache-server-01,6379/TCP,Critical
```

## Next Steps

1. Complete this inventory for all servers
2. Upload CSV to Azure Migrate
3. Review and validate imported data
4. Proceed with assessment phase
