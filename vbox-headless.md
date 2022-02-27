# Starting VirtualBox VMs in headless mode

## List VMs

```
VBoxManage list vms
```

## Start VMs

```
VBoxManage startvm "Alpine Linux v3.15 Ansible Controller" --type headless
VBoxManage startvm "Alpine Linux v3.15 Clone Ansible Host 1" --type headless
VBoxManage startvm "Alpine Linux v3.15 Clone Ansible Host 2" --type headless
```

```
VBoxManage startvm "AlpGitea" --type headless
VBoxManage startvm "AlpGitea2" --type headless
```

## Shutdown via ACPI

```
VBoxManage controlvm "Alpine Linux v3.15 Ansible Controller" acpipowerbutton
VBoxManage controlvm "Alpine Linux v3.15 Clone Ansible Host 1" acpipowerbutton
VBoxManage controlvm "Alpine Linux v3.15 Clone Ansible Host 2" acpipowerbutton
```

```
VBoxManage controlvm "AlpGitea" acpipowerbutton
VBoxManage controlvm "AlpGitea2" acpipowerbutton
```

## Change settings

Change RAM quantity

```
VBoxManage modifyvm "Alpine Linux v3.15 Ansible Controller" --memory 512
VBoxManage modifyvm "Alpine Linux v3.15 Clone Ansible Host 1" --memory 1024
VBoxManage modifyvm "Alpine Linux v3.15 Clone Ansible Host 2" --memory 1024
```
