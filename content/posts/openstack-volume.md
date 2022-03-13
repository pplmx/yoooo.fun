---
title: OpenStack Volume
date: 2020-07-24T15:18:53+08:00
lastmod: 2020-10-23T10:37:50+08:00
slug: 4fac985f9e2437c47a11e691dfac8d4c
draft: false
categories: [openstack]
tags: [storage,cinder]
keywords: OpenStack, cinder, storage, volume
description: Update QoS by OpenStack CLi
---

# OpenStack Volume

## FYI

```bash
# list the compute nodes where the servers locate
openstack server list --long -c ID -c Name -c Host -c 'Power State' -c 'Networks'

# list the instance names that are servers' alias
openstack server show vm4qos1 -c id -c name -c 'OS-EXT-SRV-ATTR:host' -c 'OS-EXT-SRV-ATTR:instance_name'

# merge the above two command
# get some servers' instance_name, host and so on
openstack server list -c ID -c Name | \
	grep vm4qos* | \
	awk -F'|' '{ print $2 }' | \
	sed 's@^[[:space:]]*@@g;s@[[:space:]]*$@@g' | \
	xargs -n1 openstack server show -c id -c name -c addresses -c 'OS-EXT-SRV-ATTR:host' -c 'OS-EXT-SRV-ATTR:instance_name'
	
# or nova list, the same as above
# fields can get from `nova show some-vm`'s Property 
nova list --fields name,OS-EXT-SRV-ATTR:instance_name,OS-EXT-SRV-ATTR:host
nova list --fields name,OS-EXT-SRV-ATTR:instance_name,OS-EXT-SRV-ATTR:host --name vm4qos*
```



## Prepare Environment for QoS

**pre_env4qos.sh**

```bash
#!/usr/bin/env bash

# Firstly, create network
netw_id=$(openstack network create cc_net_1 -c id | awk -F'|' '{ print $3 }' | sed '1,3d;$d;s@^[[:space:]]*@@g;s@[[:space:]]*$@@g'
# neutron subnet-create --name cc_net_1_sub ${netw_id} 192.168.1.0/24
# or
openstack subnet create cc_net_1_sub --network ${netw_id} --subnet-range 192.168.1.0/24

# 1. create volume type
openstack volume type create frontend_qos_1
openstack volume type create frontend_qos_2

openstack volume type create backend_qos_1
openstack volume type create backend_qos_2

# 2.1. create a QoS-1
openstack volume qos create qos1 --consumer front-end \
--property read_iops_sec=2000 \
--property write_iops_sec=2048

# 2.2. create a QoS-111
openstack volume qos create qos111 --consumer back-end \
--property read_iops_sec=2000 \
--property write_iops_sec=2048

# 3. associate QoS and volume type
openstack volume qos associate qos1 frontend_qos_1
openstack volume qos associate qos1 frontend_qos_2

openstack volume qos associate qos111 backend_qos_1
openstack volume qos associate qos111 backend_qos_2

# 4. create volumes
openstack volume create --type frontend_qos_1 --size 1 volume01
openstack volume create --type frontend_qos_1 --size 1 volume02
openstack volume create --type frontend_qos_2 --size 2 volume03
openstack volume create --type frontend_qos_2 --size 2 volume04

openstack volume create --type backend_qos_1 --size 1 volume11
openstack volume create --type backend_qos_1 --size 1 volume12
openstack volume create --type backend_qos_2 --size 2 volume13
openstack volume create --type backend_qos_2 --size 2 volume14

# 5. create a vm
openstack server create vm4qos1 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos2 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos3 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos4 --flavor 3 --image BAT-image --nic net-id="${netw_id}"

openstack server create vm4qos11 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos12 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos13 --flavor 3 --image BAT-image --nic net-id="${netw_id}"
openstack server create vm4qos14 --flavor 3 --image BAT-image --nic net-id="${netw_id}"

# wait for creating VMs
echo "Creating VMs, please wait for 80s.>>>>>>>"
sleep 80

# 6. attach a volume for a vm
# openstack server add volume INSTANCE_ID VOLUME_ID
openstack server add volume vm4qos1 volume01
openstack server add volume vm4qos2 volume02
openstack server add volume vm4qos3 volume03
openstack server add volume vm4qos4 volume04

openstack server add volume vm4qos11 volume11
openstack server add volume vm4qos12 volume12
openstack server add volume vm4qos13 volume13
openstack server add volume vm4qos14 volume14

```

## Remove Env for QoS

**del_env4qos.sh**

```bash
#!/usr/bin/env bash

# ******** reset env ********
# delete server
openstack server delete vm4qos1
openstack server delete vm4qos2
openstack server delete vm4qos3
openstack server delete vm4qos4

openstack server delete vm4qos11
openstack server delete vm4qos12
openstack server delete vm4qos13
openstack server delete vm4qos14

# wait for deleting VMs
echo "Deleting VMs, please wait for 80s.>>>>>>>"
sleep 80

# delete volume
openstack volume delete volume01
openstack volume delete volume02
openstack volume delete volume03
openstack volume delete volume04

openstack volume delete volume11
openstack volume delete volume12
openstack volume delete volume13
openstack volume delete volume14


# delete volume type and qos
openstack volume type delete frontend_qos_1
openstack volume type delete frontend_qos_2
openstack volume type delete backend_qos_1
openstack volume type delete backend_qos_2
openstack volume qos delete qos1
openstack volume qos delete qos111

# remove network
openstack subnet list | grep cc_net_1_sub | awk -F'|' '{ print $2 }' | sed 's@^[[:space:]]*@@g;s@[[:space:]]*$@@g' | xargs -n1 openstack subnet delete

openstack network list | grep cc_net_1 | awk -F'|' '{ print $2 }' | sed 's@^[[:space:]]*@@g;s@[[:space:]]*$@@g' | xargs -n1 openstack network delete

```

## Update QoS

```bash
openstack volume qos set --property "read_iops_sec=10000" --property "write_iops_sec=8000" qos1
openstack volume qos set --property "read_bytes_sec=2000" --property "write_bytes_sec=2048" qos1
openstack volume qos unset --property "read_iops_sec" --property "write_iops_sec" qos1
openstack volume qos disassociate qos1 --volume-type frontend_qos_2
openstack volume qos disassociate qos1 --all
openstack volume qos associate qos1 frontend_qos_1
openstack volume qos associate qos1 frontend_qos_2
```
