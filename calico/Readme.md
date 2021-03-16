Scope:
    Install Calico network under kubernetes cluster
Exit criteria:
    - installed calico network with BGP routing and ipip overlay network
    - every node is separate BGP router
    - full mesh or route reflector scema is used
Solution:
    1. Download Calico manifest file (50 nodes or less)
        curl https://docs.projectcalico.org/manifests/calico.yaml -O
    2. Edit manifest file as necessary
    3. Apply manifest file:
        kubectl apply -f calico.yaml
    4. Download and install calicoctl:
        curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.17.2/calicoctl
        - add "x" attribute 
        - move calicoctl to location is in your PATH.
    5. Create BGP global configuration manifest file (bgp-conf.yaml). Define:
        - desired AS number
        - nodeToNodeMeshEnabled -> 'true' to use full mesh or 'false' to use route reflector
        - other desired params
    6. Apply bgp-conf.yaml
        calicoctl apply -f bgp-conf.yaml
    7. Define BGP peering (bgp-peer.yaml):
        - 
            peerIP: <>
            asNumber: <>
        - or via selector (add necessary labels to nodes) and use this template:
            kind: BGPPeer
            apiVersion: projectcalico.org/v3
            metadata:
              name: peer-with-route-reflectors
            spec:
              nodeSelector: all()
              peerSelector: route-reflector == 'true'
    8. Apply manifest for BGP peering:
        calicoctl apply -f bgp-peer.yaml
    
    9. Helpful command:

        calicoctl get bgpConfiguration -o wide
        calicoctl get bgpConfiguration -o yaml
        calicoctl get bgpPeer o wide
        calicoctl get bgpPeer -o yaml
        calicoctl node status (runs one the node - show current ipv4/ipv6 bgp status of this node)
        calicoctl delete bgpPeer <IP>
        calicoctl patch BGPConfig default --patch \
        '{"spec": {"serviceExternalIPs": [{"cidr": "x.x.x.x"}, {"cidr": "y.y.y.y"}]}}'

Additional task: allow ssh access to worker-nodes from certain hosts
    To secure hosts in the calico networks HostEndPoint, workloadEndpoint,GlobalNetworkPolicy, NetworkPolicy, felixConfiguration are using.
    There are two main concept as object that be secured: Host and workload.
    Host is any node in your kubernetes cluster.
    Workload is pod, container etc.
    It is needed to create HostEndpoint to enable Calico filter hosts (node) traffic for certain interface.
    Hostendpoint define interface you want to secure.
    As soon as your host endpoint is defined and applied all traffic from/to specified interface  will be droped (by default) 
    if no any global network policy defined (except proto:ports defined in failsafe).
    Failsafe chain define proto:ports combination that will be always accessable to prevent accidentally loosing connection
    to importnat service ssh, dns, dhcp etc.
    To define your own secure rule(s):
        - make and apply your own Global Network policy to define allowing / denying rules for certain services

        [do next step only if proto:port combination is included in failsafe by default]
        - exclude proto:port from felix Configuration (FailsafeInboundHostPorts/FailsafeOutboundHostPorts)
            calicoctl get felixconfiguration --export -o yaml > felix.yaml
            edit felix as it is needed
            calicoctl apply -f felix.yaml
            (restart may be required)
    You can define many policies and choose hosts to which they will be applied by selector.

    Workloadendpoint define interface you want to secure from worload perspective.
    To specify your own secure rules to filter traffic to/from workloadendpoint 
    you need to make and appy NetworkPolicy.
    You can define many policies and choose workloads they will be applied by selector.
        
    helpful command:
        calicoctl get   GlobalNetworkPolicy
                        hostendpoint
                        workloadendpoint
                        NetworkPolicy
                        felixConfiguration
        also -o yaml option could be used
    


    