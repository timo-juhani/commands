# NSO

## Installation

````
# Unpack Linux binary
sh nso.bin

# Create a NSO instance (local setup)
source ~/nso/ncsrc
ncs-setup \
--package nso/packages/neds/cisco-nx-cli-5.23 \
--package nso/packages/neds/cisco-asa-cli-6.6 \
--package nso/packages/neds/cisco-ios-cli-6.91 \
--package nso/packages/neds/cisco-iosxr-cli-7.45 \
--dest nso-instance
cd ~/nso-instance
````

## Launch NSO

```
cd ~/nso-instance
ncs

# Check NSO status
ncs —status | grep status

# Enter CLI
ncs_cli -u admin -C

# Navigation
config
top 
end
exit
```

## Devices

### Authentication Credentials

````
# Create an authentication group
devices authgroups group labadmin
# Set the credentials
default-map remote-name cisco
default-map remote-password cisco
default-map remote-secondary-password cisco
````

### Add Devices

````
# Name the device
devices device edge-sw01
# Management IP address 
address 10.10.20.172
# Authentication credentials
authgroup labadmin
# NED type
device-type cli ned-id cisco-ios-cli-6.67
# Remote command protocol
device-type cli protocol ssh
# Not the most secure way to go
ssh host-key-verification none
# Consider using SSH keys for authentication

# Ping the device
ping

# Send a connection to the device
connect

# Unlock the device because by default NSO adds them as locked
state admin-state unlocked
````

### Device Synchronization

```
# Check for sync
devices device ios1 check-sync

# Sync CDB from all inventory devices by fetching their configuration
devices sync-from
```

### Device Groups

````
# Use device groups to make working with the inventory more efficient.
# Groups are logical tags.
devices device-group SPOKES
device-name rtr01
device-name rtr02

# Group that captures all devices.
devices device-group ALL
device-group SPOKES
device-group HUBS
<etc>

# Check sync
devices device-group ALL check-sync
````

### View Configuration

````
# Device configuration
show running-config devices device internet-rtr01 config
show running-config devices device internet-rtr01 | de-select config
# Interfaces
show running-config devices device internet-rtr01 config interface
show running-config devices device internet-rtr01 config interface | display json
# Interface IP addresses
# NSO allows using wildcards in commands
show running-config devices device dist-sw01 config interface Vlan * ip address
# XML format
show configuration | display xml
show full-configuration devices device ios1 config | display xml

````

### Update Configuration

````
# Enter device configuration mode
devices device ios0 config
# Verify the path
pwd
# Enter configuration changes and commit

# Configure multiple devices simultaneously
devices device * config ios:enable password magic
````

## Commit Changes

```
# Commit dry-run
commit dry-run outformat native
# Apply changes
commit
```

## Rollback Changes

````
# See the rollback for last configuration update
show configuration rollback changes
# Load the rollback
rollback configuration
# Verify the intended rollback
show configuration
# Verify the commands that will be sent to the device
commit dry-run outformat native
# Rollback
commit
````

## Templates

Devices can be managed individually using NSO as a super-CLI but using templates
multiple devices can be managed at same time using a shared template.

### Create

````
# Create a template using a correct NED
# NOTE: If the template must support multiple OS types just add another ned-id
# and punch in the configuration. 
devices template SET-DNS-SERVER
ned-id cisco-nx-cli-5.20
config
# Add configuration items (OpenDNS servers)
ip name-server servers 208.67.222.222
ip name-server servers 208.67.220.220
# Check on the changes
show configuration
# Save the changes
commit
````

### Apply

````
# List the devices
show devices list
# Apply the template on the target device
devices device dist-sw01 apply-template template-name SET-DNS-SERVER
# Or apply using device group
devices device-group ALL apply-template template-name SET-DNS-SERVER
# Perform the pre-commit SOP
show configuration
commit dry-run outformat native
commit
# If the commit doesn't look good revert the staged changes with
revert
````

## Device Operational State

### Configuration DB

Reads from NSO's configuration DB.

```
show devices device dist-sw01 platform
show devices device dist-sw01 platform serial-number
# Remember that wildcard works
show devices device * platform serial-number
show devices device * platform version
```

### Live Status

Reads directly for the device as it runs.

````
# NED supported commands
show devices device dist-sw01 live-status port-channel
show devices device dist-sw01 live-status ip route

# Raw commands using exec flag
devices device dist-sw01 live-status exec show license usage
devices device dist-sw01 live-status exec any dir
# Pipe the output to a file on the OS
devices device dist* live-status exec show license usage | save /tmp/output.txt
````

## Configuration Compliance



## Services

### Packages

```
# Creating a package
cd ~/packages
ncs-make-package --service-skeleton template simple-service
cp ~/src/simple-service.yang ~/packages/simple-service/src/yang/simple-service.yang 
cp ~/src/template.xml ~/packages/simple-service/templates/simple-service-template.xml 
make -C ~/packages/simple-service/src
echo "request packages reload" | ncs_cli -u admin

# Check package status
show packages package oper-status
```

### Service Configuration

```
# Configure a service on a device
simple-service test1 device ios0 secret mypasswd

# Get service modifications
simple-service test1 get-modifications

# Redeploy a service if changes were made manually after deploying the service
simple-service test1 re-deploy

# Delete the service
no simple-service test1
```

## Templates

### Variable syntax

<secret>{./secret}</secret>