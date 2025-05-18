# droplet.wiki.do

Notes and code for a federated wiki farm hosted at digital ocean.

1. Add a [Reserved IP](https://docs.digitalocean.com/products/networking/reserved-ips/) in Digital Ocean's console
2. create ssh configuration and key pairs
  `wiki-wiki-keygen IP_ADDR` (use the Reserved IP address)
  Two pairs of keys (two private keys and their corresponding public keys) will be generated. Save passphrases for the private keys where you can remember them.
3. create a new [docker droplet](https://marketplace.digitalocean.com/apps/docker) using the Digital Ocean template
  I chose the smallest droplet available in the same datacenter as the
  Reserved IP.  Add a new SSH Key in the Digital Ocean form using the
  contents of the root key you generated in step 2:
  `~/.ssh/wiki-wiki/root-id_ecdsa.pub`
  I also added improved metrics and alerting
4. Once the droplet is created, assign the Reserved IP to the new droplet.
5. Double-check the ssh configuration and root key:
  `ssh bootstrap-wiki-wiki -- echo Hello World`
6. Bootstrap the VM with several customizations
    ssh-add ~/.ssh/wiki-wiki/root-id_ecdsa  # to reduce passphrase re-entry
    bootstrap/remote-config create-user-account
    bootstrap/remote-config configure-firewall
    bootstrap/remote-config cleanup-after-one-click
    ssh-add -d ~/.ssh/wiki-wiki/root-id_ecdsa  # to resume requiring passphrase
7. Double-check the ssh configuration and harvester key:
  `ssh wiki-wiki -- echo Hello World`
8. Generate an encryption key for runtime secrets
    ssh-add ~/.ssh/wiki-wiki/harvester-id_ecdsa  # to reduce passphrase re-entry
    bootstrap/remote-keygen CONFIRM`
