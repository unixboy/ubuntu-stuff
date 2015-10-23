Public static ip for vagrant box::

config.vm.network :bridged
config.vm.provision :shell, :path => "vagrant-setup.sh"

vagrant-setup.sh

#!/bin/bash

ip="188.120.244.5/24"
gateway=""
dns="8.8.8.8"


#####################
# NEW VM
#####################
if [ ! -f /etc/network/if-up.d/custom-network-config ]; then

cat >/etc/network/if-up.d/custom-network-config <<EOL
#!/bin/bash
if [ "\$IFACE" != "eth1" ]; then
exit 0
fi
ifconfig eth1 down
ifconfig eth1 ${ip} up
route del default
route add default gw ${gateway} dev eth1
service apache2 restart
EOL

cat >/etc/resolv.conf <<EOL
nameserver ${dns}
EOL

chmod +x /etc/network/if-up.d/custom-network-config
/etc/init.d/networking restart


#####################
# EXISTING VM
#####################
else

/etc/init.d/networking restart

fi
