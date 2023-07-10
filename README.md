# docker-demo-network

## bridge

```bash
git clone https://github.com/azarov-serge/docker-demo-network.git
cd docker-demo-network
docker bild -t demo-network

docker run --name=network-1 -d demo-network:latest
docker run --name=network-2 -d demo-network:latest

docker inspect bridge
```

Show IP (Containers)

```json
[
	{
		"Name": "bridge",
		"Id": "caa34bd5617adda80c2f49b645862f337c9201fd8102e48a00343d8dc62dae03",
		"Created": "2023-07-10T10:12:38.430719066Z",
		"Scope": "local",
		"Driver": "bridge",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": null,
			"Config": [
				{
					"Subnet": "172.17.0.0/16",
					"Gateway": "172.17.0.1"
				}
			]
		},
		"Internal": false,
		"Attachable": false,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
			"2d545bd62620136024cfcf430f69485c9295368b888e4fea2792650556f4a451": {
				"Name": "network-2",
				"EndpointID": "cad7058183282939a97f1038b2ba934c845b4a00bbbd6d465410a9c5cd967c6e",
				"MacAddress": "02:42:ac:11:00:05",
				"IPv4Address": "172.17.0.5/16",
				"IPv6Address": ""
			},
			"3de76a7c4052883fbf311cc8b284d7896dd7d124a7612666d9e46e4c571b2789": {
				"Name": "network-1",
				"EndpointID": "14407719c04857a73b5a61c2b5c622d212fe99f7746807b634c8d80187af7003",
				"MacAddress": "02:42:ac:11:00:04",
				"IPv4Address": "172.17.0.4/16",
				"IPv6Address": ""
			}
		},
		"Options": {
			"com.docker.network.bridge.default_bridge": "true",
			"com.docker.network.bridge.enable_icc": "true",
			"com.docker.network.bridge.enable_ip_masquerade": "true",
			"com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
			"com.docker.network.bridge.name": "docker0",
			"com.docker.network.driver.mtu": "1500"
		},
		"Labels": {}
	}
]
```

Enter container network-1

```bash
docker exec -it network-1 sh
```

Inside container network-1 run

```bash
curl 172.17.0.5:3000
```

Response:

```bash
{"eth0":["172.17.0.5"]}
```

Enter container network-2

```bash
docker exec -it network-2 sh
```

Inside container network-2 run

```bash
curl 172.17.0.4:3000
```

Response:

```bash
{"eth0":["172.17.0.4"]}
```

If enter container name

```bash
curl network-1:3000
```

Error:

```bash
curl: (6) Could not resolve host: network-1
```

Create own bridge network

```bash
docker network create hide-network
docker network ls
```

List:

```bash
NETWORK ID     NAME           DRIVER    SCOPE
caa34bd5617a   bridge         bridge    local
db97a4a25849   hide-network   bridge    local
d249472ed3f9   host           host      local
653e29a62c13   none           null      local
```

Add containers to own network

```bash
docker network connect hide-network network-1
docker network connect hide-network network-2

docker network inspect hide-network
```

hide-network data (Containers)

```json
[
	{
		"Name": "hide-network",
		"Id": "db97a4a258494b6c9f36afce103cd4b25377a4bfe5d0780e9b849ff5a4c427f4",
		"Created": "2023-07-10T14:30:17.268817859Z",
		"Scope": "local",
		"Driver": "bridge",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": {},
			"Config": [
				{
					"Subnet": "172.18.0.0/16",
					"Gateway": "172.18.0.1"
				}
			]
		},
		"Internal": false,
		"Attachable": false,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
			"2d545bd62620136024cfcf430f69485c9295368b888e4fea2792650556f4a451": {
				"Name": "network-2",
				"EndpointID": "9b01062ba78f840f0eba991dfab833091a0d834a3f2e66f0701014814e52b650",
				"MacAddress": "02:42:ac:12:00:03",
				"IPv4Address": "172.18.0.3/16",
				"IPv6Address": ""
			},
			"3de76a7c4052883fbf311cc8b284d7896dd7d124a7612666d9e46e4c571b2789": {
				"Name": "network-1",
				"EndpointID": "4334a61c1206a3da8a82317bd4d0c1ba200970a9e8d2631d353bc6d29d330d4e",
				"MacAddress": "02:42:ac:12:00:02",
				"IPv4Address": "172.18.0.2/16",
				"IPv6Address": ""
			}
		},
		"Options": {},
		"Labels": {}
	}
]
```

Enter container network-2

```bash
docker exec -it network-2 sh
```

Inside container network-2 run

```bash
curl network-1:3000
```

Response:

```bash
{"eth0":["172.17.0.4"],"eth1":["172.18.0.2"]}
```

Add new network-3

```bash
docker run --name=network-3 --network=hide-network -d demo-network:latest
docker inspect bridge
```

bridge data (network-3 is not exist)

```json
[
	{
		"Name": "bridge",
		"Id": "caa34bd5617adda80c2f49b645862f337c9201fd8102e48a00343d8dc62dae03",
		"Created": "2023-07-10T10:12:38.430719066Z",
		"Scope": "local",
		"Driver": "bridge",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": null,
			"Config": [
				{
					"Subnet": "172.17.0.0/16",
					"Gateway": "172.17.0.1"
				}
			]
		},
		"Internal": false,
		"Attachable": false,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
			"2d545bd62620136024cfcf430f69485c9295368b888e4fea2792650556f4a451": {
				"Name": "network-2",
				"EndpointID": "cad7058183282939a97f1038b2ba934c845b4a00bbbd6d465410a9c5cd967c6e",
				"MacAddress": "02:42:ac:11:00:05",
				"IPv4Address": "172.17.0.5/16",
				"IPv6Address": ""
			},
			"3de76a7c4052883fbf311cc8b284d7896dd7d124a7612666d9e46e4c571b2789": {
				"Name": "network-1",
				"EndpointID": "14407719c04857a73b5a61c2b5c622d212fe99f7746807b634c8d80187af7003",
				"MacAddress": "02:42:ac:11:00:04",
				"IPv4Address": "172.17.0.4/16",
				"IPv6Address": ""
			}
		},
		"Options": {
			"com.docker.network.bridge.default_bridge": "true",
			"com.docker.network.bridge.enable_icc": "true",
			"com.docker.network.bridge.enable_ip_masquerade": "true",
			"com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
			"com.docker.network.bridge.name": "docker0",
			"com.docker.network.driver.mtu": "1500"
		},
		"Labels": {}
	}
]
```

```bash
docker inspect hide-network
```

hide-network data (network-3 is exist)

```json
[
	{
		"Name": "hide-network",
		"Id": "db97a4a258494b6c9f36afce103cd4b25377a4bfe5d0780e9b849ff5a4c427f4",
		"Created": "2023-07-10T14:30:17.268817859Z",
		"Scope": "local",
		"Driver": "bridge",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": {},
			"Config": [
				{
					"Subnet": "172.18.0.0/16",
					"Gateway": "172.18.0.1"
				}
			]
		},
		"Internal": false,
		"Attachable": false,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
			"2d545bd62620136024cfcf430f69485c9295368b888e4fea2792650556f4a451": {
				"Name": "network-2",
				"EndpointID": "9b01062ba78f840f0eba991dfab833091a0d834a3f2e66f0701014814e52b650",
				"MacAddress": "02:42:ac:12:00:03",
				"IPv4Address": "172.18.0.3/16",
				"IPv6Address": ""
			},
			"3de76a7c4052883fbf311cc8b284d7896dd7d124a7612666d9e46e4c571b2789": {
				"Name": "network-1",
				"EndpointID": "4334a61c1206a3da8a82317bd4d0c1ba200970a9e8d2631d353bc6d29d330d4e",
				"MacAddress": "02:42:ac:12:00:02",
				"IPv4Address": "172.18.0.2/16",
				"IPv6Address": ""
			},
			"ea47ef805f46a6bd38de2f07c5677ef241b0574cbc710f055ab7bbe70c447188": {
				"Name": "network-3",
				"EndpointID": "c3e91a400c99cf8d00145f58044865049b6edacf677bc9da057c88576d5f2dda",
				"MacAddress": "02:42:ac:12:00:04",
				"IPv4Address": "172.18.0.4/16",
				"IPv6Address": ""
			}
		},
		"Options": {},
		"Labels": {}
	}
]
```

Add new network-4

```bash
docker run --name=network-4 -p 3000:3000 --network=hide-network -d demo-network:latest
docker ps
```

List:

```bash
49e9502289c1   demo-network:latest      "docker-entrypoint.s…"   56 seconds ago   Up 54 seconds   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   network-4
ea47ef805f46   demo-network:latest      "docker-entrypoint.s…"   9 minutes ago    Up 9 minutes                                                network-3
2d545bd62620   demo-network:latest      "docker-entrypoint.s…"   46 minutes ago   Up 46 minutes                                               network-2
3de76a7c4052   demo-network:latest      "docker-entrypoint.s…"   46 minutes ago   Up 46 minutes                                               network-1
```

## host

Add new network-5

```bash
docker run --name=network-5 -d --network host demo-network:latest
docker ps
```

List:

```bash
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS     NAMES
58b024fd927b   demo-network:latest   "docker-entrypoint.s…"   56 seconds ago   Up 55 seconds             network-5
```

Inspect host

```bash
docker network inspect host
```

host data (Containers)

```json
[
	{
		"Name": "host",
		"Id": "d249472ed3f9bb7639d25f8f5c52ffe443c63eb9c48ad444f7c1f9e0bcd19cb3",
		"Created": "2023-07-06T09:27:27.698909491Z",
		"Scope": "local",
		"Driver": "host",
		"EnableIPv6": false,
		"IPAM": {
			"Driver": "default",
			"Options": null,
			"Config": []
		},
		"Internal": false,
		"Attachable": false,
		"Ingress": false,
		"ConfigFrom": {
			"Network": ""
		},
		"ConfigOnly": false,
		"Containers": {
			"58b024fd927bf96638aec18182c640e310ca343b1b4247cbb039aebac9977922": {
				"Name": "network-5",
				"EndpointID": "67a6edd90e4d0f922070dba6ddad06c5961a9f4feba8e7e07e96cbe245078deb",
				"MacAddress": "",
				"IPv4Address": "",
				"IPv6Address": ""
			}
		},
		"Options": {},
		"Labels": {}
	}
]
```

Run curl

```bash
curl 127.0.0.1:3000
```

Response:

```bash
{"enp0s3":["10.0.2.15"]}
```

## null

Add new network-6

```bash
docker run --name=network-6 -d --network none demo-network:latest
docker ps
```

List:

```bash
CONTAINER ID   IMAGE                 COMMAND                  CREATED              STATUS              PORTS     NAMES
ff47eb194137   demo-network:latest   "docker-entrypoint.s…"   About a minute ago   Up About a minute             network-6
58b024fd927b   demo-network:latest   "docker-entrypoint.s…"   8 minutes ago        Up 8 minutes                  network-5
```

Enter container network-6

```bash
docker exec -it network-6 sh
```

Inside container network-6 run

```bash
curl localhost:3000
```

Response:

```bash
{}
```

## dns

```bash
docker run --name=network-9 -d --dns 8.8.8.8 demo-network:latest
docker exec -it network-9 sh
```

Inside container network-9 run

```bash
cat /etc/resolv.conf
```

Result:

```bash
nameserver 8.8.8.8
```
