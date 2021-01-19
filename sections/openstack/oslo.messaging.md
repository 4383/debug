# "No calling threads waiting for msg_id"

https://github.com/openstack/oslo.messaging/blob/master/oslo_messaging/_drivers/amqpdriver.py#L414

if you see that in the logs, then increasing rpc_response_timeout can definitely help
that is the error message you get if you've hit rpc_response_timeout and then
the reply *does* eventually come back, just late
