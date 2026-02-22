DC Hostname: dc1
DC IP Address: 192.168.1.XXX # Replace with the actual IP address of the DC container
FS Hostname: fs1
FS IP Address: 192.168.1.XXX # Replace with the actual IP address of the FS container
Main DNS Address: 192.168.1.XXX # Replace with the actual IP address of the main DNS server
AD Domain FQDN: example.local # Replace with your actual AD domain FQDN
Host Network Interface: eth0 # Replace with the actual network interface name on the host machine
DOMAIN_DN: DC=example,DC=local # Replace with the distinguished name form of your AD domain (derived from AD Domain FQDN)
SUBNET: 192.168.1.0/24 # Replace with the subnet used by the DC/FS containers
GATEWAY: 192.168.1.1 # Replace with the default gateway IP address for the SUBNET
CONTAINER_IP_RANGE: 192.168.1.128/25 # Replace with the IP range within SUBNET reserved for Docker containers