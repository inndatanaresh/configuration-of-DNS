# DNS Configuration in CentOS

## **Step 1: Install Required Packages**
```bash
yum install bind bind-utils -y
```

## **Step 2: Configure named.conf**
Edit the named configuration file:
```bash
vi /etc/named.conf
```
Add the master IP and subnet mask.

Also, add the zone entries for forward and reverse lookup.

---

## **Step 3: Create Forward Zone File**
Edit or create the forward zone file:
```bash
vi /var/named/forward.unixmen
```

**Content of the Forward Zone File:**
```bash
$TTL 86400
@   IN  SOA     masterdns.inndata.local. root.unixmen.local. (
        2011071001  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          masterdns.inndata.local.
@       IN  A           10.10.5.82
@       IN  A           10.10.5.83
masterdns       IN  A   10.10.5.82
clint           IN  A   10.10.5.83
dns             IN  A   10.10.5.72
ns              IN  A   10.10.5.80
```

---

## **Step 4: Create Reverse Zone File**
Edit or create the reverse zone file:
```bash
vi /var/named/reverse.unixmen
```

**Content of the Reverse Zone File:**
```bash
TTL 86400
@   IN  SOA     masterdns.inndata.local. root.unixmen.local. (
        2011071001  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          masterdns.inndata.local.
@       IN  PTR         clint.inndata.local.
@       IN  PTR         dns.inndata.internal.
@       IN  PTR         ns.inndata.internal.
masterdns       IN  A   10.10.5.82
client          IN  A   10.10.5.83
82     IN  PTR         masterdns.inndata.local.
83     IN  PTR         clint.inndata.local.
72     IN  PTR         dns.inndata.internal.
80     IN  PTR         ns.inndata.internal.
```

---

## **Step 5: Start and Enable DNS Service**
```bash
systemctl enable named
systemctl start named
```

## **Step 6: Firewall Configuration**
```bash
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp
firewall-cmd --reload
```

---

## **Step 7: Set Permissions**
```bash
chgrp named -R /var/named
chown -v root:named /etc/named.conf
restorecon -rv /var/named
restorecon /etc/named.conf
```

---

## **Step 8: Verify DNS Configuration**
```bash
named-checkconf /etc/named.conf
named-checkzone unixmen.local /var/named/forward.unixmen
named-checkzone unixmen.local /var/named/reverse.unixmen
```

---

## **Step 9: Configure resolv.conf**
Edit the resolver configuration:
```bash
vi /etc/resolv.conf
```
Add the following line:
```bash
nameserver 192.168.1.101
```
Restart the network service:
```bash
systemctl restart network
```

---

## **Step 10: Testing DNS Server**
Test name resolution using ping:
```bash
ping master.inndata.in
ping slave.inndata.in
```

---

## **Slave Configuration**
1. Add the master IP in:
   - `/etc/resolv.conf`
   - `/etc/sysconfig/network-scripts/ifcfg-eth0`

2. Add all slave and client networks in the master’s **/etc/hosts** file.

---

## **Conclusion**
This setup ensures proper DNS resolution with forward and reverse zone configurations, allowing reliable hostname resolution across the network.
