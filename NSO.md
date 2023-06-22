# NSO

## Installation

## Launch NSO

```
ncs

# Check NSO status
ncs —status | grep status

# Enter CLI
ncs_cli -u admin -C
```

## Device Synchronization

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