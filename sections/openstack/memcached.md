# Memcached


## Which service use memcached on RHOSP

Example of command to retrieve which service use memcached on RHOSP:

```
[root@controller-0 puppet-generated]# grep -irH 11211
heat/etc/heat/heat.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
heat_api_cfn/etc/heat/heat.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
heat_api/etc/heat/heat.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
glance_api/etc/glance/glance-api.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
cinder/etc/cinder/cinder.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
horizon/etc/openstack-dashboard/local_settings:#        'LOCATION': '127.0.0.1:11211',
horizon/etc/openstack-dashboard/local_settings:        'LOCATION': [ '172.17.1.117:11211','172.17.1.110:11211','172.17.1.14:11211', ],
memcached/etc/sysconfig/memcached:PORT="11211"
keystone/etc/keystone/keystone.conf:#memcache_servers = localhost:11211
neutron/etc/neutron/neutron.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
nova_metadata/etc/nova/nova.conf:#memcache_servers=localhost:11211
nova_metadata/etc/nova/nova.conf:memcache_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
nova_metadata/etc/nova/nova.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
nova/etc/nova/nova.conf:#memcache_servers=localhost:11211
nova/etc/nova/nova.conf:memcache_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
nova/etc/nova/nova.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
placement/etc/placement/placement.conf:memcached_servers=172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
swift/etc/swift/object-expirer.conf:memcache_servers = 172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
swift/etc/swift/proxy-server.conf:memcache_servers = 172.17.1.117:11211,172.17.1.110:11211,172.17.1.14:11211
```
