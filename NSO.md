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
````

### Device Synchronization

```
# Check for sync
devices device ios1 check-sync

# Sync CDB from all inventory devices
devices sync-from
```

## Configuration

### Set Level
```
# Global
config
# Device
devices device ios0 config
```

### Change

````
# Configure multiple devices simultaneously
devices device * config ios:enable password magic
````

### View

````
# Native
show full-configuration devices device ios1 config
# XML
show full-configuration devices device ios1 config | display xml
# Configuration changes during this session
show configuration
````

## Commit Changes

```
# Commit dry-run
commit dry-run outformat native
# Apply changes
commit
```

## Navigation

````
# Return top level context
top
# Return from config mode
end
````

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