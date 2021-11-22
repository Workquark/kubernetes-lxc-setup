# RKE Installation - 

### Create key on host VM - 

```
cat <public_key_file_on_host> | lxc exec <container> -- sh -c "cat >> /home/ubuntu/.ssh/authorized_keys"

Eg : cat id_rsa.pub | lxc exec rke -- sh -c "cat >> /home/ubuntu/.ssh/authorized_keys"
```

### Install ssh daemon into container 

```
https://github.com/artefactcorp/ssh2lxd
```