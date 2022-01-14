```
sudo iptables -S |grep DROP| sed 's/-A/-D/' >rules  # -A becomes -D: delete

#vi rules  # check that everything is correct

#cat rules | while read line; do iptables $line; done


iptables-save | awk '/^[*]/ { print $1 }
                     /^:[A-Z]+ [^-]/ { print $1 " ACCEPT" ; }
                     /COMMIT/ { print $0; }' | iptables-restore

#All policies will be reset to ACCEPT as well as flushing every table in current use.

#All chains other than the built in chains will no longer exist.
```

or 

```
#!/usr/bin/env bash
set -eu
declare -A chains=(
    [filter]=INPUT:FORWARD:OUTPUT
    [raw]=PREROUTING:OUTPUT
    [mangle]=PREROUTING:INPUT:FORWARD:OUTPUT:POSTROUTING
    [security]=INPUT:FORWARD:OUTPUT
    [nat]=PREROUTING:INPUT:OUTPUT:POSTROUTING
)
for table in "${!chains[@]}"; do
    echo "${chains[$table]}" | tr : $"\n" | while IFS= read -r; do
        iptables -t "$table" -P "$REPLY" ACCEPT
    done
    iptables -t "$table" -F
    iptables -t "$table" -X
done
```
