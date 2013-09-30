nDeploy
======

a tiny, simple Flask-based application designed to provide automated os installs

best used with ipxe as follows:

- compile your own ipxe binary (`make bin/undionly.kpxe EMBED=ndeploy.ipxe`),
  where ndeploy.ipxe looks something like:

```
#!ipxe
dhcp
echo
echo attempting to netboot with IP ${ip}
echo
set ndeploy_server ndeploy.your.domain
chain http://${ndeploy_server}/provisions/${uuid}/boot ||
echo
echo provision not found or net boot failed, booting ipxe shell
echo
shell
```

- everything after ${ndeploy_server} must be as above

- setup your DHCP insfrastructure to serve undionly.kpxe to all pxe boot requests

- set the boot order on all your servers to pxe boot first

- provision some boot sequences and go go go!

usage guidance
--------------

- keep your binary serving http infrastructure somewhere else

- make sure you create some .ipxe and .ks files (by hand) in the templates directory before 
  doing anything. The provisioning process requires them and the web UI is populated from them.

- check out the sample files in the templates directory for some pointers

- check out provision.json in the templates directory for an idea of the data model

- the web UI is just javascript talking to the backend API

- the web UI only gives you 3 options for a build sequence, but you can create as many as you want via curl:

```
curl -i -H "Content-Type: application/json" -X POST -d \
'{
"uuid":"b9ae67b8-25e7-4633-b46a-8702c4bf1d34",
"fqdn":"ndeploy-test.local",
"os_template":"centos_6.4.ks",
"boot_sequence":{"1":"firmware_1.ipxe","2":"firmware_2.ipxe","3":"centos_6.4.ipxe","4":"local.ipxe"}
}' \
http://[ndeploy.fqdn]/provisions/
```

URLs
---
**/** - the web UI

**/provisions/** - list of uuids for which provisions have been created

**/provisions/{uuid}** - individual provision.json

**/provisions/{uuid}/reprovision** - to quickly restore a provision to its original state `curl -X POST http://{ndeploy.fqdn}/provisions/{uuid}/reprovision`

**/templates/** - list of valid templates for use in the os_template and boot_sequence fields of a provision

**/unprovisioned/** - list of uuids for which ipxe phoned home but no provision was found

todo
----

- harden up (more defensive code, real error handling)
- build a network and regional settings information service
- finish off get_network_details and get_regional_settings
- ~~node.js~~ gevent rewrite
