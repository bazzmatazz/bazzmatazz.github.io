---
layout: post
title:  "Virtual Box Find IP Address From CommandLine"
date:   2020-06-08 01:00:23 +0500
categories: 
---

Setup SSH Connection On A New Box:

```
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh
sudo ufw allow ssh
```

List all VMs:
`VBoxManage list vms`

Should return something like:
```
"sandboxlinux" {XXXXXXXXXXXXXXXXXXXXXXXXXXXX}
"server" {XXXXXXXXXXXXXXXXXXXXXXXXXXXX}
```

- Download extension pack after checking your vbox version
  - VBoxManage --version
  - If latest download from:
    - https://www.virtualbox.org/wiki/Downloads
  - For older builds:
    - https://www.virtualbox.org/wiki/Download_Old_Builds_5_2
  
- Install the extension pack
  - `VBoxManage extpack install --replace ~/Downloads/tools/vboxextensionpack/Oracle_VM_VirtualBox_Extension_Pack-5.2.6.vbox-extpack`

- Run the VM:
  - `VBoxManage startvm server --type headless`

- Find ip:
  - `VBoxManage guestproperty enumerate server`

- Find running VMs:
  - `VBoxManage list runningvms`
`

### Restore Snapshot
- `vboxmanage snapshot server18 list`
- `vboxmanage snapshot server18 restore server18_withssh`