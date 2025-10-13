# Enable nested vt-x/amd-v virtualbox
linux -> terminal ->
```terminal
VBoxManage modifyvm <VirtualMachineName> --nested-hw-virt on
```
windows -> install folder -> cmd -> 
```cmd
VBoxManage modifyvm <YourVirtualMachineName> --nested-hw-virt on 
```
