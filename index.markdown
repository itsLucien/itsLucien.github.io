---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
```bash
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