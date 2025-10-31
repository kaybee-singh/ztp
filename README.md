Create the KVM host for baremetal
```bash
sudo qemu-img create -f qcow2 /home/karan/nvme2/sno02.qcow2 200G
export HUB_CLUSTER_NAME="hubztp"
export CLUSTER_NAME="sno01"
export UUID="deed1e55-fe11-f0e5-0dd5-babb1ed1a010"
export MAC="00:00:00:00:00:14"
```
```bash
sudo virt-install   --name=${HUB_CLUSTER_NAME}-${CLUSTER_NAME}   --uuid=${UUID}   --ram=65536   --vcpus=16   --cpu host-passthrough   --os-type linux   --os-variant rhel8.0   --noreboot   --events on_reboot=restart   --noautoconsole   --boot hd,cdrom   --import   --disk path=/home/karan/nvme2/sno02.qcow2,size=200   --network  network=default,mac=00:00:00:00:00:14,model=virtio
```
```bash
oc create ns ai-sno-cluster001
oc create secret generic ai-bmh-secret \
  -n ai-sno-cluster001 \
  --from-literal=username="admin" \
  --from-literal=password="redhat"
  oc create secret generic pull-secret   -n ai-sno-cluster001   --from-file=.dockerconfigjson=./pull-secret   --type=kubernetes.io/dockerconfigjson
```
add agentconfig
```bash
cat agent-service.config 
apiVersion: agent-install.openshift.io/v1beta1
kind: AgentServiceConfig
metadata:
  name: agent
spec:
  databaseStorage:
    storageClassName: nfs-client   # your new SC
    accessModes: ["ReadWriteOnce"]
    resources:
      requests:
        storage: 10Gi
  filesystemStorage:
    storageClassName: nfs-client
    accessModes: ["ReadWriteOnce"]
    resources:
      requests:
        storage: 20Gi
```
Configure 
```bash
apiVersion: siteconfig.open-cluster-management.io/v1alpha1
kind: ClusterInstance
metadata:
  name: abc 
  namespace: abc 
spec:
  cpuPartitioningMode: None
  clusterName: abc 
  clusterImageSetNameRef: img4.19.17-x86-64-appsub
  machineNetwork:
    - cidr: 192.168.4.0/22
  networkType: OVNKubernetes
  sshPublicKey: ssh-ed25519 AAAAC3NzaC1lZDI1
  nodes:
    - automatedCleaningMode: disabled
      nodeNetwork:
        config:
          dns-resolver:
            config:
              search:
                - balk.ocp.local
              server:
                - 192.168.7.126
          interfaces:
            - ipv4:
                dhcp: true
                enabled: true
              name: ens1s0
              state: up
              type: ethernet
          routes:
            config:
              - destination: 0.0.0.0/0
                next-hop-address: 192.168.122.1
                next-hop-interface: ens1s0
                table-id: 254 
        interfaces:
          - macAddress: '00:00:00:00:00:14'
            name: ens1s0
      bmcCredentialsName:
        name: ai-bmh-secret
      ironicInspect: ''
      hostName: abc 
      bootMode: UEFI
      role: master
      bootMACAddress: '00:00:00:00:00:14'
      templateRefs:
        - name: ai-node-templates-v1
          namespace: open-cluster-management
      cpuArchitecture: x86_64
      bmcAddress: 'redfish-virtualmedia+http://192.168.122.1:8000/redfish/v1/Systems/deed1e55-fe11-f0e5-0dd5-babb1ed1a010'
  baseDomain: balk.ocp.local
  holdInstallation: false
  templateRefs:
    - name: ai-cluster-templates-v1
      namespace: open-cluster-management
  cpuArchitecture: x86_64
  pullSecretRef:
    name: pull-secret
```
