# droplet-wiki-wiki

Notes and code for a federated wiki farm hosted at digital ocean.


## A. create an ssh config and related keys
1. Add a [Reserved IP](https://docs.digitalocean.com/products/networking/reserved-ips/) in Digital Ocean's console
2. create ssh configuration and key pairs
  `wiki-wiki-keygen IP_ADDR` (use the Reserved IP address)
  Two pairs of keys (two private keys and their corresponding public keys) will be generated. Save passphrases for the private keys where you can remember them.

## B. create the digital ocean droplet with a custom user account
1. create a new [docker droplet](https://marketplace.digitalocean.com/apps/docker) using the Digital Ocean template
  I chose the smallest droplet available in the same datacenter as the
  Reserved IP.  Add a new SSH Key in the Digital Ocean form using the
  contents of the root key you generated in step 2:
  `~/.ssh/wiki-wiki/root-id_ecdsa.pub`
  I also added improved metrics and alerting
2. Once the droplet is created, assign the Reserved IP to the new droplet.
3. Double-check the ssh configuration and root key:
  `ssh bootstrap-wiki-wiki -- echo Hello World`
4. Bootstrap the VM with several customizations
   ``` bash
    ssh-add ~/.ssh/wiki-wiki/root-id_ecdsa  # to reduce passphrase re-entry
    bootstrap/remote-config create-user-account
    bootstrap/remote-config configure-firewall
    bootstrap/remote-config cleanup-after-one-click
    ssh-add -d ~/.ssh/wiki-wiki/root-id_ecdsa  # to resume requiring passphrase
    ```
5. Double-check the ssh configuration and harvester key:
  `ssh wiki-wiki -- echo Hello World`

## C. create and publish the deploy configuration
(we encrypt the secrets which needs another pair of keys)
1. Generate an encryption key for runtime secrets
    ``` bash
    ssh-add ~/.ssh/wiki-wiki/harvester-id_ecdsa  # to reduce passphrase re-entry
    bootstrap/remote-keygen CONFIRM`
    ```
2. Create the base configuration
   (use your own name, your own email address for Let's Encrypt, and good place to save the configuration)
   ``` bash
   # config/create-config AUTHOR LE_EMAIL CONFIG_FILE
   config/create-config "Old MacDonald" "o.macdonald@example.com" ../config-wiki-wiki/wiki.example.com.json
   ```
3. Add a domain name to the wiki config
   (this is a step that you'll do again when you add a new wiki to your farm)
   ``` bash
   config/add-subdomain ../config-wiki-wiki/wiki.example.com.json wiki.example.com
   ```
4. Publish the wiki config
   I'm using GitHub Pages to publish my config. This lets me use version control for my configuration. You could too, from a fork of the config repo.
   ``` bash
   pushd ../config-wiki-wiki
   git add .
   git commit -m "added wiki.example.com"
   git push
   popd
   ```

## D. deploy the wiki
``` bash
deploy/first-ever CONFIG_URL  # replace CONFIG_URL with the URL from wherever you published the wiki configuration
```
