# Vagrant

## Plugins

```text
vagrant plugin install vagrant-vbguest
# vagrant vbguest --status
# vagrant vbguest --do install
vagrant plugin install vagrant-reload
```

## Get box Storage Controller name

```text
cat ~/.vagrant.d/boxes/bento-VAGRANTSLASH-ubuntu-18.04/202010.24.0/virtualbox/box.ovf | grep -i storagecontroller
```

Output example.

```text
<StorageControllers>
  <StorageController name="IDE Controller" type="PIIX4" PortCount="2" useHostIOCache="true" Bootable="true"/>
  <StorageController name="SATA Controller" type="AHCI" PortCount="1" useHostIOCache="false" Bootable="true" IDE0MasterEmulationPort="0" IDE0SlaveEmulationPort="1" IDE1MasterEmulationPort="2" IDE1SlaveEmulationPort="3">
  </StorageController>
</StorageControllers>
```

