# eksworkshop

<details>
  <summary>zeal</summary>
  
  ```
  kubernetes -> CRI -> crio -> OCI -> runc
  (OCI - Open contaniner initiative)
  
  Hight level runtime: (pull image from regstry, unpack to continer root fs, generate OCI runtime json, launch OCI runtime: runsc) 
  docker
  contaninerd
  cri-o
  podman
  
  Low level runtime: (OCI)
  runc   (run container process: need config.json, rootfs)
  
  ```
  ```
  gVisor
  dmesg (diagnosit message)
  
  ```
  ```
  capsh --print   # display the capability. if not installed, apt install libcap2-bin
  
  checkov -f pod.yml
  
  
  # /var/log/pods/...   # pod log
  
  sysdig proc.name=vi or proc.name=cat
  sysdig proc.name=cat and container.id!=host
  sysdig -c spy_users   # chisel
  sysdig -cl       # list chisel
  sysdig -h     # help
  sysdig -l     # list field
  csysdig
  
  output: demo %evt.time %user.uid %proc.name
  timeout 25s falcon | grep demo
  
  cat log.txt | awk '{print $4 $5 $6}'
  cat log.txt | awk '{print $4 " " $5 " " $6}
  
systemctl enable falco
systemctl start falco
journalctl -fu falco
falco  # manual run
  
  
  ```
  
</details>
