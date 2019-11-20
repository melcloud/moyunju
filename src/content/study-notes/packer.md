---
title: "Packer Study Note"
date: 2018-06-06T10:44:45+10:00
tags: [
    "packer",
    "hashicorp",
    "devOps",
    "immutable image",
    "study note"
]
menu: "Study Notes"
---

## Terminologies
- **Artifacts**. The result of a single build. Can be an ID (AMI) or files (Azure VHD, Virtualbox ovf)
- **Builds**. Tasks to produce artifacts
- **Builders**. Components of Packer which is used in in Build
- **Provisioners**. Components of Packer that install and configure software within a running machine prior to that machine being turned into a static image
- **Post-processors**. Components of Packer that take the result of a builder or another post-processor and process that to create a new artifact. E.g. upload file, compress file

## Template

Defined in JSON file. A basic template contains variables, builders, provisioners and post-processors.

```json
{
  "description": "my template",
  "min_packer_version": "",
  "variables": {
  },
  "_comment": "This is a comment",
  "builders": [{
  }],
  "provisioners": [{
  }],
  "post-processors": [{
  }]
}
```

Only root level key can be commented by using `_comment`

## CLI

- `packer -autocomplete-install` installs auto completion for commands. This also applies to other Hashicorp tools.
- `packer validate example.json` is used to validate template syntax.
- `packer build -var 'aws_access_key=YOUR ACCESS KEY' -var 'aws_secret_key=YOUR SECRET KEY' example.json` is used to build a template. variables can be passed in by *-var*. It is recommended to set secret through environment variables rather passed in through cli. On Windows, **Single quotes need to be replaced with double quotes**. Use `-only=x,y,z` and `-except=x,y,z` to include/exclude build targets
- Multiple builds can be defined which Packer will build them in parallel. Use `-parallel=false` to disable


## Machine readable output

`packer -machine-readable <command>` will produce machine readable output in the format of `timestamp,target,type,data1,data2`. More information [here](https://www.packer.io/docs/commands/index.html#format-for-machine-readable-output)

## Recipes

### Linux

Provisioning based on user input

``` json
{
  "type": "shell-local",
  "command": "if [ ! -z \"{{user `do_nexpose_scan`}}\" ]; then python -u trigger_nexpose_scan.py; fi"
}
```

Set environment variable as user variable

``` json
{
  "variables": {
    "home": "{{env `HOME`}}"
  }
}
```

### Windows
In cloud environment, the following script can be used to enable WinRM connection (Copied from [Packer document](https://www.packer.io/intro/getting-started/build-image.html)):

```powershell
# Set administrator password
net user Administrator SuperS3cr3t!
wmic useraccount where "name='Administrator'" set PasswordExpires=FALSE

# First, make sure WinRM can't be connected to
netsh advfirewall firewall set rule name="Windows Remote Management (HTTP-In)" new enable=yes action=block

# Delete any existing WinRM listeners
winrm delete winrm/config/listener?Address=*+Transport=HTTP  2>$Null
winrm delete winrm/config/listener?Address=*+Transport=HTTPS 2>$Null

# Create a new WinRM listener and configure
winrm create winrm/config/listener?Address=*+Transport=HTTP
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="0"}'
winrm set winrm/config '@{MaxTimeoutms="7200000"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service '@{MaxConcurrentOperationsPerUser="12000"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/client/auth '@{Basic="true"}'

# Configure UAC to allow privilege elevation in remote shells
$Key = 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System'
$Setting = 'LocalAccountTokenFilterPolicy'
Set-ItemProperty -Path $Key -Name $Setting -Value 1 -Force

# Configure and restart the WinRM Service; Enable the required firewall exception
Stop-Service -Name WinRM
Set-Service -Name WinRM -StartupType Automatic
netsh advfirewall firewall set rule name="Windows Remote Management (HTTP-In)" new action=allow localip=any remoteip=any
Start-Service -Name WinRM
```

However, it is recommended to use OpenSSH for this now. See James Nugent's post [here](https://operator-error.com/2018/04/16/windows-amis-with-even/).
