# Driver Signing Instructions

Use these instructions when the Ethernet adapter stops working after a BIOS update.

After an update, you should just need to re-enroll the MOK.

## Generate Signing Key

One time. Generate signing keys

```shell
cd /var/lib/shim-signed/mok
sudo openssl req -new -x509 -newkey rsa:2048 \
    -keyout MOK.priv -out MOK.pem -nodes -days 36500 \
    -subj "/CN=Custom Module Signer/"
sudo openssl x509 -outform DER -in MOK.pem -out MOK.der
```

## Install Key in BIOS

Enroll signing key in BIOS. Might need to be done after any BIOS update. I think so, but I'm not sure.

1. Import the signing key (public key)

```shell
sudo mokutil --import MOK.der
```

2. Reboot
3. Enroll MOK key

## Sign the driver

Should only need to be done if the driver was re-built.

```shell
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 \
    /var/lib/shim-signed/mok/MOK.priv \
    /var/lib/shim-signed/mok/MOK.der \
    /lib/modules/$(uname -r)/kernel/drivers/net/ethernet/realtek/r8126.ko
```
