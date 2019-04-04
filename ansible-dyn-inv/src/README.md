# Ansible Meetup in Zürich - Demo

## What we do
1. How to create a cross cloud iventory group group having
   - all debian-9 servers:
   - webservers

2. Cleanup all servers with a dynamic inventory afterwards.

## Requirements:

* Ansible >=2.8
* The enviroment variables for tokens of Vultr and Cloudscale has been set.
* Some Virtual machines with prefix `web-` has been deployed on CloudScale and Vultr.

# Demo

Querying the API, see we have servers:
~~~sh
http --json https://api.vultr.com/v1/server/list "API-Key:$VULTR_API_KEY"
http --json https://api.cloudscale.ch/v1/servers "Authorization:Bearer $CLOUDSCALE_API_TOKEN"
~~~

See the docs about the dynamic inventories:
~~~sh
ansible-doc -t inventory -l
ansible-doc -t inventory vultr
~~~

Graph and list the inventory of cloudscale servers:
~~~sh
cat hosts/cloudscale.yml
ansible-inventory  -i hosts/cloudscale.yml --graph
ansible-inventory  -i hosts/cloudscale.yml --list
~~~

Do the same for vultr servers:
~~~sh
cat hosts/vultr.yml
ansible-inventory  -i hosts/vultr.yml --graph
ansible-inventory -i hosts/vultr.yml --list
~~~

We transform the value of key os into a group, in vultr:
~~~
"os": "Debian 9 x64 (stretch)",
~~~

The corresponding section in `hosts/vultr.yml`
~~~yaml
keyed_groups:
  - separator: ""
    key: os | lower | regex_replace(' x64.*','')
~~~

For cloudscale we just need to lower a value of key `cloudscale.image.slug`:

~~~yaml
keyed_groups:
  - separator: ""
    key: cloudscale.image.slug | lower
~~~

Result can be determined by running:
~~~sh
ansible-inventory -i hosts/cloudscale.yml -i hosts/vultr.yml --graph
~~~

Cleanup all

~~~sh
ansible-playbook cloud-cleanup.yml -i hosts/vultr.yml -i hosts/cloudscale.yml
~~~
