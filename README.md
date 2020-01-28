
# CDH OS Best Practice

## List
- Hostname Resolution
  ```
  dig <hostname>
  dig –x <ip_address_returned_from_hostname_lookup)
  ```
  ```
  dig themis.apache.org
  themis.apache.org.
  1758
  IN
  A
  140.211.11.105

  dig -x 140.211.11.105
  105.11.211.140.in-addr.arpa. 3513 IN

  PTR
  themis.apache.org.
  ```

- Functional Accounts
- NTP
  ```
  systemctl start ntpd.service
  systemctl enable ntpd.service
  ```
- Disable SELinux
  ```
  /etc/selinux/config
  SELINUX=disabled
  ```
- IPv6  
Hadoop does not support IPv6. IPv6 configurations should be removed, and IPv6-related services should be stopped.  

- Disable firewalld
  ```
  systemctl stop firewalld.service
  systemctl disable firewalld.service
  ```
- Remove/Disable these services
  ```
  bluetooth
  cups
  iptables
  ip6tables
  postfix
  ```
  ```
  systemctl list-unit-files --type service | grep enabled
  ```
- Process Memory  
A minimum of 4 GB of memory should be reserved on all nodes for operating system and other non-Hadoop use

- OS Tuning
  - Disable the tuned Service
    ```
    tuned-adm off
    tuned-adm list
    ```
    ```
    No current active profile
    ```
    ```
    systemctl stop tuned
    systemctl disable tuned
    ```
  - Disabling Transparent Hugepages (THP)  
    ```
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
    ```
    ```
    chmod +x /etc/rc.d/rc.local
    ```
    - Add the following line to the GRUB_CMDLINE_LINUX options in the /etc/default/grub file:
    ```
    transparent_hugepage=never
    ```
    ```
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```
  - Setting the vm.swappiness
    ```
    sudo sysctl -w vm.swappiness=1
    ```
  - Decrease Reserve Space
    ```
    tune2fs -l /dev/sde1 | egrep "Block size:|Reserved block count"
    Reserved block count:  36628312
    Block size:            4096
    ```
    Cloudera recommends reducing the root user block reservation from 5% to 1% for the DataNode volumes. To set reserved space to 1% with the tune2fs command:
    ```
    tune2fs -m 1 /dev/sde1
    ```
  - Networking Parameters  
    The following parameters are to be added to /etc/sysctl.conf to optimize various network behaviors.
    ```
    net.ipv4.tcp_timestamps=0
    net.ipv4.tcp_sack=1
    net.core.netdev_max_backlog=250000

    net.core.rmem_max=4194304
    net.core.wmem_max=4194304
    net.core.rmem_default=4194304
    net.core_wmem_default=4194304
    net.core.optmem_max=4194304

    net.ipv4.tcp_rmem=4096 87380 4194304
    net.ipv4.tcp_wmem=4096 65536 4194304

    net.ipv4.tcp_low_latency=1
    net.ipv4.tcp_adv_win_scale=1
    ```

## Filesystem
- Creation Options
   ```
   mkfs –t ext4 –m 1 –O -T largefile \
   sparse_super,dir_index,extent,has_journal /dev/sdb1
   ```
- Disk Mount Options
  In /etc/fstab
  ```
  /dev/sda1 / ext4 noatime 0 0
  ```
- Disk Mount Naming Convention
  ```
  /dn01
  /dn02
  .
  .
  ./dn0n
  ```
