##Linux 交换分区

```bash
cat /proc/swaps
dd if=/dev/zero of=/mnt/swap bs=1M count=512
mkswap /mnt/swap
chmod 600 /mnt/swap
swapon /mnt/swap
free
echo '/mnt/swap               swap                    swap    defaults        0 0' >> /etc/fstab
```