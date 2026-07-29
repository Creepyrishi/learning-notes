ZRAM and EarlyOOM

**systemctl**

```bash
sudo systemctl start ssh
sudo systemctl status ssh
sudo systemctl stop ssh
```

systemclt is a interface which uses systemd to manage background process and control system state. there are 

**systemd** is the standard init system and system manager used by most modern Linux distributions to bootstrap user space and manage system processes. It is the very first process that starts after the Linux kernel boots (PID 1), acting as the parent or ancestor of all other processes running on the system.

we can manage some task using start, stop, restart and relaod commands.

```bash
sudo systemctl stop ssh
```

we  can also do auto start at boot using enable and disable command

```bash
sudo systemctl enable nginx
sudo systemctl disable nginx 
sudo systemctl enable --now nginx
```

we can also use `poweroff` and `reboot` command via systemctl. and it also provie us way to monitor the services and see the failuers

