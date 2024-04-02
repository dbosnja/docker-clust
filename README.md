# docker-cluster

Some steps I tried:
Locally, I'm in the docker group so no need for sudo, also dc is alias for /usr/bin/docker, so *dc = sudo docker*

1. Build the Consul image, this image is going to be used by every host/node.
	`dc build -t dbosnja/consul .`
	
2. Run the Consul Server with one node:
	`dc run -p 8500:8500 -p 54:53/udp -h server dbosnja/consul -server -bootstrap`
	
![image](https://user-images.githubusercontent.com/26286601/225943931-f0dec49b-9fd7-4c31-9910-fb2aebe1afe1.png)


Now I wanted to set up the Consul cluster with 2 nodes, these 2 nodes would represent
imaginary two machines on 2 different place in the world.
For some reasons I believed I had to have docker installed on them -> 
3. build the node image
```
cd cluster-nodes/
dc build -t dbosnja/node .
dc run -it -h nodeA --name nodeA -v /var/run/docker.sock:/var/run/docker.sock dbosnja/node -> first node
dc run -it -h nodeB --name nodeB -v /var/run/docker.sock:/var/run/docker.sock dbosnja/node -> the second
```
4. Now the first node would be the cluster leader and the 2nd would be a follower:
```
root@nodeA:/# PUBLIC_IP=$(hostname -i)
root@nodeA:/# echo $PUBLIC_IP
172.17.0.3
root@nodeB:/# PUBLIC_IP=$(hostname -i)
root@nodeB:/# JOIN_IP=172.17.0.3
```
5. Now start the Consul cluster on nodeA, this is why I thought I need docker inside a docker container

```
root@nodeA:/# docker run -d -h $HOSTNAME -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8510:8500 -p 63:53/udp \
--name nodeA_agent dbosnja/consul -server -advertise $PUBLIC_IP -bootstrap-expect 2

root@nodeA:/# docker logs nodeA_agent
==> WARNING: Expect Mode enabled, expecting 2 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'nodeA'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 53, RPC: 8400)
      Cluster Addr: 172.17.0.3 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2023/03/17 14:52:12 [INFO] raft: Node at 172.17.0.3:8300 [Follower] entering Follower state
    2023/03/17 14:52:12 [INFO] serf: EventMemberJoin: nodeA 172.17.0.3
    2023/03/17 14:52:12 [INFO] consul: adding LAN server nodeA (Addr: 172.17.0.3:8300) (DC: dc1)
    2023/03/17 14:52:12 [INFO] serf: EventMemberJoin: nodeA.dc1 172.17.0.3
    2023/03/17 14:52:12 [INFO] consul: adding WAN server nodeA.dc1 (Addr: 172.17.0.3:8300) (DC: dc1)
    2023/03/17 14:52:12 [ERR] agent: failed to sync remote state: No cluster leader
    2023/03/17 14:52:13 [WARN] raft: EnableSingleNode disabled, and no known peers. Aborting election.
==> Newer Consul version available: 1.15.1
    2023/03/17 14:52:33 [ERR] agent: failed to sync remote state: No cluster leader
    2023/03/17 14:52:39 [ERR] agent: coordinate update error: No cluster leader
root@nodeA:/# 
```
6. Repeat step 5 for the 2nd node:
```
root@nodeB:/# docker run -d -h $HOSTNAME -p 8355:8300 -p 8311:8301 -p 8321:8301/udp -p 8312:8302 -p 8313:8302/udp -p 8410:8400 -p 8520:8500 -p 73:53/udp \ 
						--name nodeB_agent dbosnja/consul -server -advertise $PUBLIC_IP -join $JOIN_IP
4d0daac522f93e1225c5340b1328ae6bc8928d355fe40a2f91f8f750c86a616a
root@nodeB:/# docker logs nodeB_agent
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Joining cluster...
==> dial tcp 172.17.0.3:8301: getsockopt: connection refused
```

