---
layout: page
title: Code
permalink: /code/
---
## **Code**
This page is where I keep any code snippets that I work on. Complete projects can be found on my github or on their own individual pages on this site.

Currently I have programs made in C++, Rust, and in Bash.
### **C++**  
>Shortcut Trace  
Here is a C++ function I have found useful when working with Windows files. The function takes a parameter of the full path of a Windows Shortcut as a wstring and returns the path to the executable that the shortcut points to.
```C++
#include <string>
#include <iostream>
#include <filesystem>
#include <windows.h>
#include <shobjidl_core.h>
#include <objidl.h>
#include <shlobj.h>

std::string getShortcutParentPath(std::wstring pathShortcut)
{
		
	IShellLinkA* psl;
	CoInitialize(0); 
	char* tempStr = new char[MAX_PATH];
	std::wstring path = pathShortcut;

	HRESULT hr = CoCreateInstance(CLSID_ShellLink, NULL, CLSCTX_INPROC_SERVER, IID_IShellLink, (LPVOID*)&psl);

	if (SUCCEEDED(hr))
	{
		IPersistFile* ppf;
		hr = psl->QueryInterface(IID_IPersistFile, (LPVOID*)&ppf);
		if (SUCCEEDED(hr))
		{
			hr = ppf->Load(path.c_str(), STGM_READ);

			if (SUCCEEDED(hr))
			{
				WIN32_FIND_DATA wfd;
				psl->GetPath(tempStr, MAX_PATH, &wfd, SLGP_UNCPRIORITY | SLGP_RAWPATH);
			}
		}
	}
	std::string message = tempStr;
	fs::path sPath(message);
	message = sPath.parent_path().string();
	return message;		
}
```
### **Rust**
```Rust
```
### **Bash**  

>Usage Warning  
>This is a shell script I wrote for a school assignment. The program takes input on capacity (optional) and email addresses (required). When the program is ran, if the disk usage is over the amount specified in the parameters or 60 if unspecified, the email-addresses listed in the parameters will receive and internal mail message stating the drives are near capacity. 
   
Syntax:
```console
$ usage-warning -c 75 email-address address1 address2 address3
```
```Bash
#!/usr/bin/ksh

CRITICAL=90
CAPACITY=60
Emails=

send_warnings () {
    FILESYSTEM=${1% *}
    USEPCENT=${1##* }
    USEPCENT=${USEPCENT%\%*}
    USAGELINE=`df -k -t nfs -t ext4 -t ext2 | grep "$FILESYSTEM\\|Filesystem"`
    SUBJECTLINE=
    if [ $USEPCENT -ge $CRITICAL ]
    then
    SUBJECTLINE="Critical Warning: the file system $FILESYSTEM is greater than or
equal to 90% capacity"
    for email in $Emails
    do
    echo "$USAGELINE" | mailx -s "$SUBJECTLINE" "$email"
    done
    elif [ $USEPCENT -ge $CAPACITY ]
    then
    SUBJECTLINE="Warning: the file system $FILESYSTEM is above $CAPACITY % used"
    for email in $Emails
    do
    echo "$USAGELINE" | mailx -s "$SUBJECTLINE" "$email"
    done
    fi
}

while [ $# -gt 0 ]
    do
    case $1 in
    -c)
        shift
        CAPACITY=$1
        ;;
    email-address)
        shift
        if [ $# -eq 0 ]
        then
        echo "Error: no emails specified, exiting program"
        exit 1
        fi
        Emails=$@
        ;;
    esac
    if [ $# -gt 0 ]
    then
    shift
    fi
    done
output=$(df -k -t nfs -t ext4 -t ext2 --output=source,pcent | sed 1d)
while IFS= read -r line; do
    send_warnings "$line"
done <<< "$output"
```