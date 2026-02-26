# Encrypted DNS docker stack
---
This repository is intended as a demonstration of how I acheived end to end ecrypted DNS with pihole dns blocking. It is worth mentioning that this arrangement does not guarantee encrypted dns, since port 53 (standard DNS) is open. If you want to only provide encrypted DNS, then port 53 will have to be disabled by removing port 53 from the pihole section of the dns docker compose file. The webUI of pihole is also unencrypted and should only be exposed to secure connections(like limiting access to localhost and using an ssh tunnel to access the webUI).  In case it is not obvious, this project does not conform to best practices. **Use at your own risk.**


That being said, if you choose to follow along with my demonstration, you can clone this repo to your own server, grab a TLS cerficate using the lego docker container and finally start the DNS container stack. But first, there are a few things in these files that will need to be changed.


## Required Changes

---

### Lego docker compose file

The lego docker compose file will require several modifications

1. The certificates volume will need to be changed. The path on the left side of the colon will need to be changed to something appropriate for your system. The change will likely look something like this

`- /home/hwhl/cert:/.lego/cerfificates`    --->   `- /home/yourname/cert:/.lefo/certificates`

2. The environment variables will need to be changed to your cloudflare information. The placeholders are surrounded by <brackets>. You will need to crate an API token with your cloudflare account. Make sure you also remove <>.

3. You will also need to change the place holders in the "command" line of the compoose file. The email used here is for LetsEncrypt, and does not have to be associated with your cloudflare account. <yourdomain> is the full domain of the server using this certificate. In my example I used 'dns.hwhl-demo.xyz'. The last place holder is <domainzone> and it is your domain without any subdomains. Using the same example as before, my domain zone is hwhl-demo.xyz.

Once these changes are done, you can use lego to get a TLS certificate from LetsEncrypt. You do that by running `docker compose up`  from the lego directory that holds the docker compose file for lego. Once the container has run, it will shut down.

---

### DNS stack docker compose file

1. The second volume for the dnsproxy container will neet to be changed. The left side of the colon will need to match the lego docker compose file as above. Using the example above it should end up looking like this:

`- home/yourname/cert:/cert`

2. The TZ environment variable for the pihole container should be changed to match your time zone. If you don't know what your time zone is, then you can check this [list](https://gist.github.com/adamgen/3f2c30361296bbb45ada43d83c1ac4e5).

### The DNS proxy config file

Both the tls-crt: and tls-key: lines will need to be changed to reflect the full domain of your DNS server. This will have to match the files grabbed by lego. If you are unsure check the contents in the cert directory created by the lego docker container. The result should look like this:

`tls-crt: '/cert/dns.hwhl-demo.xyz.crt'`   --->   `tls-crt: '/cert/dns.yourdomain.com.crt'`
`tls-key: '/cert/dns.hwhl-demo.xyz.key'`   --->   `tls-key: '/cert/dns.yourdomain.com.key'`

With these final changes the DNS stack can be started by running ` docker compose up -d ` from the directory that holds the compose file for the dns stack.

## If Pihole container wont start

If you cannot start the pihole container and are getting an error message that says "the address is already in use", then pihole container is likely in conflict with systemd-resolved stub listener. Using your preferred text editor, you will need to change /etc/systemd/resolved.conf. Find the line that says:
```
#DNSStubListener=yes
```
Remove the # and change the yes to no. The final line should like this:
```
DNSStubListener=no
```
Then restart the systemd-resolved.service to load the new configuration file. You can do that with this command:
```
sudo systemctl restart systemd-resolved.service
```

Now there should not be a conflict when starting the pihole container. Happy DNS blocking!
