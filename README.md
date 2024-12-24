
# README

This document outlines the procedures for working with NIM (Network Installation Manager) environments, specifically focusing on SPOT (Shared Product Object Tree) instances and maintenance tasks.

## What is SPOT?
SPOT is a fundamental resource in the NIM environment. It provides the necessary `/usr` file system and network boot support for clients. It includes:
- AIX kernel
- Executable commands
- Libraries and applications

## Sources for Creating SPOT Instances
1. **ISO Images**: Use the same server type as the affected server.
2. **MKSYSB Backup**: Use a backup from the same server type.

### Using ISO Image as `lpp_source` to Create a SPOT Instance
1. Download a similar version of the base ISO image from IBM Entitled System Support.
2. Save the image to the NIM server.
3. Define the `lpp_source` resource:
   ```bash
   nim -o define -t lpp_source -a server=master -a location=<lpp_source_location> -a source=<ISO_image_location> -a packages=all <lpp_source_name>
   ```
4. Verify the resource using:
   ```bash
   lsnim -c resources
   lsnim -l <resource_name>
   ```
5. Create the SPOT instance:
   ```bash
   nim -o define -t spot -a server=master -a location=<spot_files_location> -a source=<lpp_source_name> <spot_name>
   ```

### Using MKSYSB Backup to Create a SPOT Instance
1. Create an MKSYSB backup from a client server.
   - AIX:
     ```bash
     mksysb -i <backup_dir/backup_file_name>
     ```
   - VIO:
     ```bash
     /usr/ios/cli/ioscli backupios -mksysb -file <mksysb_name> -nomedialib
     ```
2. Define the SPOT instance:
   ```bash
   nim -o define -t spot -a source=<mksysb_name> -a server=master -a location=<spot_files_location> <spot_name>
   ```
3. Verify the resource using:
   ```bash
   lsnim -c resources
   lsnim -l <spot_name>
   ```

### Allocate the SPOT Instance to the Affected Server
1. Allocate the SPOT instance using:
   ```bash
   nim -o maint_boot -a spot=<spot_name> <server_name>
   ```

## Maintenance Procedure
### Prerequisites
1. A non-autoinstall MKSYSB or AIX bootable media.
2. A configured NIM server with SPOT.

### Steps
1. Log in to the HMC GUI and CLI.
2. Shut down the system and activate it in SMS mode.
3. Navigate through the SMS menu to select the appropriate network boot options.
4. Access the root volume group and perform required maintenance tasks.

### Common Commands
- Change password:
  ```bash
  passwd
  ```
- Backup and modify `/etc/filesystems`:
  ```bash
  cp /etc/filesystems /etc/filesystems_backup
  sed -e "s/^\/backupmksys_tsm/*&/" /etc/filesystems > /tmp/filesystems_modified
  cp /tmp/filesystems_modified /etc/filesystems
  sync; sync; reboot
  ```

---

**Note**: Always ensure you have proper backups and permissions before performing these operations.
