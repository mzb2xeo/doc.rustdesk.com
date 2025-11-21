---
title: Client Deployment
weight: 400
pre: "<b>2.4. </b>"
---
### The most suitable deployment approach ultimately depends on your specific use case, technical expertise, and organizational requirements.
---
#### ___Support the project by purchasing a Pro license and get access to the [custom client generator](https://rustdesk.com/docs/en/self-host/client-configuration/#1-custom-client-generator-pro-only-basic-plan-or-custom-plan)___
---
### Steps to deploy using the below scripts: 
1. [Mannually configure the client](https://rustdesk.com/docs/en/self-host/client-configuration/#2-manual-config) 
2. [Export the server config string](https://rustdesk.com/docs/en/self-host/client-configuration/#3-setup-using-import-or-export) (Steps 1-4).
3. Copy the required script to an editor.   
    a. Set the $rustdesk_pw=()         
    b. Set the $rustdesk_cfg=(Config string)
4. Run on remote device.
5. Verify remote device connects. 

## PowerShell

```powershell
# Run as administrator

$ErrorActionPreference= 'silentlycontinue'

# Creates a random password.  Comment out to not use it. 
# $rustdesk_pw=(-join ((65..90) + (97..122) | Get-Random -Count 12 | % {[char]$_}))

# ___USING BOTH rustdeck_pw= variables will cause the script to fail. 

# Set the permanent password here. (8 characters or more, one upper, one lower, and a number). Note: It wont show up at the end of the script. 
$rustdesk_pw="Password1@"

# Get your server config string from your configured client or web protal
$rustdesk_cfg="Config string goes here"

################################## Edit carefully below this line #########################################
# Run as administrator and stay in the current directory
if (-Not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator))
{
    if ([int](Get-CimInstance -Class Win32_OperatingSystem | Select-Object -ExpandProperty BuildNumber) -ge 6000)
    {
        Start-Process PowerShell -Verb RunAs -ArgumentList "-NoProfile -ExecutionPolicy Bypass -Command `"cd '$pwd'; & '$PSCommandPath';`"";
        Exit;
    }
}
# Function to get the latest release information
function Get-LatestRustDeskRelease {
    $latestReleaseUri = 'https://api.github.com/repos/rustdesk/rustdesk/releases/latest'
    try {
        $releaseData = Invoke-RestMethod -Uri $latestReleaseUri -UseBasicParsing
    } catch {
        Write-Error "Failed to retrieve latest release data: $_"
        return
    }
    
    # Extract the version and download link from the JSON response
    $version = $releaseData.tag_name
    if ($version -eq "") {
        Write-Output "ERROR: Version not found."
        Exit
    }

    foreach ($asset in $releaseData.assets) {
        if ($asset.name -like "*x86_64.exe") {
            $downloadLink = $asset.browser_download_url
            break
        }
    }
    
    if (-not $downloadLink) {
        Write-Output "ERROR: Download link not found."
        Exit
    }

    # Create a custom object to return the results
    $result = [PSCustomObject]@{
        Version     = $version
        DownloadLink = $downloadLink
    }

    return $result
}

# Get the installed version of RustDesk
$rdver = ((Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\RustDesk\").Version)

# Get the latest release information from GitHub
$RustDeskOnGitHub = Get-LatestRustDeskRelease

# Compare the installed version with the web version
if ($rdver -eq $RustDeskOnGitHub.Version) {
    Write-Output "RustDesk $rdver is the newest version."
} else {
    # Create a temp directory if it doesn't exist
    if (!(Test-Path C:\Temp)) {
        New-Item -ItemType Directory -Force -Path C:\Temp | Out-Null
    }

    cd C:\Temp

    # Download the latest version of RustDesk
    Invoke-WebRequest $RustDeskOnGitHub.Downloadlink -Outfile "rustdesk.exe"

    # Silent install the downloaded executable
    Start-Process .\rustdesk.exe --silent-install

    # Wait for 20 seconds to ensure the installation completes
    Start-Sleep -seconds 20
}
# Assuming $ServiceName is defined somewhere in your script
Start-Service $ServiceName
Start-Sleep -seconds 5
$arrService.Refresh()
cd $env:ProgramFiles\RustDesk\
.\rustdesk.exe --get-id | Write-Output -OutVariable rustdesk_id
.\rustdesk.exe --config $rustdesk_cfg
.\rustdesk.exe --password $rustdesk_pw
Write-Output "..............................................."
Write-Output "RustDesk ID: $rustdesk_id"
Write-Output "Password: $rustdesk_pw"
Write-Output "..............................................."
```

## Windows batch/cmd
 - This script does NOT have the ability to generate a randomized password 

```bat
@echo off
REM Variables

set rustdesk_pw="Password1@"

REM  Get your config string from Client > Network > ID/Relay server > Export (copy icon)
set rustdesk_cfg="config string goes here"

setlocal EnableDelayedExpansion

REM Create Temp directory if it doesn't exist
if not exist C:\Temp\ md C:\Temp\
cd C:\Temp\

REM Function to get the latest release from GitHub
for /f "delims=," %%A in ('curl -s https://api.github.com/repos/rustdesk/rustdesk/releases/latest ^| findstr tag_name') do set latest_release=%%A
REM Remove quotes and leading 'v' if present
set latest_release=%latest_release:"=%
set latest_release=%latest_release:v=%

REM Download the latest release executable
curl -L "https://github.com/rustdesk/rustdesk/releases/download/%latest_release%/rustdesk-%latest_release%-x86_64.exe" -o rustdesk.exe

REM Execute RustDesk installer silently
rustdesk.exe --silent-install
timeout /t 20

REM Install the service (assuming this is for Windows)
cd "C:\Program Files\RustDesk\"
rustdesk.exe --install-service
timeout /t 20

REM Get RustDesk ID
for /f "delims=" %%i in ('rustdesk.exe --get-id ^| more') do set rustdesk_id=%%i

REM Set configuration and password
rustdesk.exe --config %rustdesk_cfg%
rustdesk.exe --password %rustdesk_pw%
echo ...............................................
REM Show the value of the ID Variable
echo RustDesk ID: %rustdesk_id%
REM Show the value of the Password Variable
echo Password: %rustdesk_pw%
echo ...............................................
```

## MSI Package installation 
- The MSI package supports command line parameters for silent installation.
- Please see [MSI package configuration](https://rustdesk.com/docs/en/client/windows/msi/)


## Winget Package Manager  
 - Run a PowerShell command from within the console or through a script, such as a Group Policy Object (GPO).
 - The syntax for installing RustDesk is as follows:
```
winget install --id=RustDesk.RustDesk  -e
```

## Linux

```sh
#!/bin/bash
apt-get update && apt-get upgrade -y
# Update the password
rustdesk_pw=(Password1@)

# Get your server config string from your configured client
rustdesk_cfg=(Config-string-here)

################################## Please Do Not Edit Below This Line #########################################

# Check if the script is being run as root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root."
    exit 1
fi

# Identify OS
if [ -f /etc/os-release ]; then
    . /etc/os-release
    OS=$NAME
    VER=$VERSION_ID
    UPSTREAM_ID=${ID_LIKE,,}
    if [ "${UPSTREAM_ID}" != "debian" ] && [ "${UPSTREAM_ID}" != "ubuntu" ]; then
        UPSTREAM_ID="$(echo ${ID_LIKE,,} | sed s/\"//g | cut -d' ' -f1)"
    fi
elif type lsb_release >/dev/null 2>&1; then
    OS=$(lsb_release -si)
    VER=$(lsb_release -sr)
elif [ -f /etc/lsb-release ]; then
    . /etc/lsb-release
    OS=$DISTRIB_ID
    VER=$DISTRIB_RELEASE
elif [ -f /etc/debian_version ]; then
    OS=Debian
    VER=$(cat /etc/debian_version)
elif [ -f /etc/SuSE-release ]; then
    OS=SuSE
    VER=$(cat /etc/SuSE-release)
elif [ -f /etc/redhat-release ]; then
    OS=RedHat
    VER=$(cat /etc/redhat-release)
else
    OS=$(uname -s)
    VER=$(uname -r)
fi

# Install RustDesk (dynamically fetch latest release)
echo "Installing RustDesk"

get_latest_release_url() {
    repo="$1"
    pattern="$2"
    curl -s "https://api.github.com/repos/$repo/releases/latest" | \
        grep "browser_download_url" | \
        grep "$pattern" | \
        cut -d '"' -f 4
}

if [ "${ID}" = "debian" ] || [ "$OS" = "Ubuntu" ] || [ "$OS" = "Debian" ] || [ "${UPSTREAM_ID}" = "ubuntu" ] || [ "${UPSTREAM_ID}" = "debian" ]; then
    url=$(get_latest_release_url "rustdesk/rustdesk" "x86_64.deb")
    if [ -z "$url" ]; then
        echo "Could not find latest RustDesk .deb package."
        exit 1
    fi
    wget "$url" -O rustdesk-latest.deb
    apt-get install -fy ./rustdesk-latest.deb > /dev/null
elif [ "$OS" = "CentOS" ] || [ "$OS" = "RedHat" ] || [ "$OS" = "Fedora Linux" ] || [ "${UPSTREAM_ID}" = "rhel" ] || [ "$OS" = "Almalinux" ] || [[ "$OS" == Rocky* ]] ; then
    url=$(get_latest_release_url "rustdesk/rustdesk" "x86_64.rpm")
    if [ -z "$url" ]; then
        echo "Could not find latest RustDesk .rpm package."
        exit 1
    fi
    wget "$url" -O rustdesk-latest.rpm
    yum localinstall ./rustdesk-latest.rpm -y > /dev/null
else
    echo "Unsupported OS"
    exit 1
fi

# Run the rustdesk command with --get-id and store the output in the rustdesk_id variable
rustdesk_id=$(rustdesk --get-id)

# Apply new password to RustDesk
rustdesk --password $rustdesk_pw &> /dev/null

rustdesk --config $rustdesk_cfg

systemctl restart rustdesk

echo "..............................................."
if [ -n "$rustdesk_id" ]; then
    echo "RustDesk ID: $rustdesk_id"
else
    echo "Failed to get RustDesk ID."
fi

echo "Password: $rustdesk_pw"
echo "..............................................."
```

## macOS Bash

```sh
#!/bin/bash

# Assign a random value to the password variable
rustdesk_pw=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 8 | head -n 1)

# Get your config string from your Web portal and Fill Below
rustdesk_cfg="configstring"

################################## Please Do Not Edit Below This Line #########################################

# Check if the script is being run as root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root."
    exit 1
fi

# Identify OS
if [ -f /etc/os-release ]; then
    # freedesktop.org and systemd
    . /etc/os-release
    OS=$NAME
    VER=$VERSION_ID

    UPSTREAM_ID=${ID_LIKE,,}

    # Fallback to ID_LIKE if ID was not 'ubuntu' or 'debian'
    if [ "${UPSTREAM_ID}" != "debian" ] && [ "${UPSTREAM_ID}" != "ubuntu" ]; then
        UPSTREAM_ID="$(echo ${ID_LIKE,,} | sed s/\"//g | cut -d' ' -f1)"
    fi

elif type lsb_release >/dev/null 2>&1; then
    # linuxbase.org
    OS=$(lsb_release -si)
    VER=$(lsb_release -sr)
elif [ -f /etc/lsb-release ]; then
    # For some versions of Debian/Ubuntu without lsb_release command
    . /etc/lsb-release
    OS=$DISTRIB_ID
    VER=$DISTRIB_RELEASE
elif [ -f /etc/debian_version ]; then
    # Older Debian, Ubuntu, etc.
    OS=Debian
    VER=$(cat /etc/debian_version)
elif [ -f /etc/SuSE-release ]; then
    # Older SuSE etc.
    OS=SuSE
    VER=$(cat /etc/SuSE-release)
elif [ -f /etc/redhat-release ]; then
    # Older Red Hat, CentOS, etc.
    OS=RedHat
    VER=$(cat /etc/redhat-release)
else
    # Fall back to uname, e.g. "Linux <version>", also works for BSD, etc.
    OS=$(uname -s)
    VER=$(uname -r)
fi

# Install RustDesk

echo "Installing RustDesk"
if [ "${ID}" = "debian" ] || [ "$OS" = "Ubuntu" ] || [ "$OS" = "Debian" ] || [ "${UPSTREAM_ID}" = "ubuntu" ] || [ "${UPSTREAM_ID}" = "debian" ]; then
    wget https://github.com/rustdesk/rustdesk/releases/download/1.2.6/rustdesk-1.2.6-x86_64.deb
    apt-get install -fy ./rustdesk-1.2.6-x86_64.deb > null
elif [ "$OS" = "CentOS" ] || [ "$OS" = "RedHat" ] || [ "$OS" = "Fedora Linux" ] || [ "${UPSTREAM_ID}" = "rhel" ] || [ "$OS" = "Almalinux" ] || [ "$OS" = "Rocky*" ] ; then
    wget https://github.com/rustdesk/rustdesk/releases/download/1.2.6/rustdesk-1.2.6-0.x86_64.rpm
    yum localinstall ./rustdesk-1.2.6-0.x86_64.rpm -y > null
else
    echo "Unsupported OS"
    # here you could ask the user for permission to try and install anyway
    # if they say yes, then do the install
    # if they say no, exit the script
    exit 1
fi

# Run the rustdesk command with --get-id and store the output in the rustdesk_id variable
rustdesk_id=$(rustdesk --get-id)

# Apply new password to RustDesk
rustdesk --password $rustdesk_pw &> /dev/null

rustdesk --config $rustdesk_cfg

systemctl restart rustdesk

echo "..............................................."
# Check if the rustdesk_id is not empty
if [ -n "$rustdesk_id" ]; then
    echo "RustDesk ID: $rustdesk_id"
else
    echo "Failed to get RustDesk ID."
fi

# Echo the value of the password variable
echo "Password: $rustdesk_pw"
echo "..............................................."
```
