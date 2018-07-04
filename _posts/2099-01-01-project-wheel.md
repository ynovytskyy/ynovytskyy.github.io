

virt-install \
  --name ubuntu-containership \
  --ram 2048 --vcpus 1 \
  --os-type linux \
  --graphics vnc,password=cantbemepty,listen=0.0.0.0 --noautoconsole \
  --network model=virtio,bridge=br0 \
  --disk path=/var/storage/kvm/images/ubuntu-containership.qcow2,format=qcow2,bus=virtio,size=2 \
  -c /var/storage/kvm/iso/ubuntu-18.04-server-amd64.iso

virsh vncdisplay ubuntu-containership










```

sudo virt-install -n web_devel -r 512 \
  --disk path=/var/lib/libvirt/images/web_devel.img,bus=virtio,size=4 \
  -c /var/lib/libvirt/images/ubuntu-18.04-server-amd64.iso \
  --network network=default,model=virtio  -v



  Portainer

  init swarm

  docker service create \
  --name portainer \
  --publish 9000:9000 \
  --replicas=1 \
  --constraint 'node.role == manager' \
  --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
  --mount type=volume,src=portainer_data,dst=/data \
  --env VIRTUAL_HOST=portainer.run.netdevlabs.com \
  --network bridge \
  portainer/portainer \
  -H unix:///var/run/docker.sock
