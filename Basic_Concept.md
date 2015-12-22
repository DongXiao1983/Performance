### MLAG 

### LACP

### VRF 

### VRRP 

### Edge router

### STP 

### VxLAN to VLAN

### overlay VXLAN  

### VXLAN VTEP

### 802.1ad

### ECMP

### VTP  

http://www.definethecloud.net/vxlan-deep-dive/   
http://www.definethecloud.net/vxlan-deep-divepart-2/   
http://www.definethecloud.net/data-center-101-local-area-network-switching/   
https://sites.google.com/site/amitsciscozone/home/data-center/vxlan   
http://www.definethecloud.net/access-layer-network-virtualization-vn-tag-and-vepa/  

### Number of header Bytes:   
	
	  Protocol Application      Number of HeaderBytes     Total Frame Size         command   
	  802.1Q  Trunking                    4               1500 + 4 + 18 = 1522   
	  QinQ pass-through                 4 + 4             1500 + 8 + 18 = 1526    system mtu 1504    
	  MPLS VPN pass-through             4 + 4             1500 + 8 + 18 = 1526    system mtu 1508      
	  uti/l2psv3 pass-through        18 + 20 + 12         1500 +50 + 18 = 1560    system mtu 1550   


[https://learningnetwork.cisco.com/thread/39098 ](https://learningnetwork.cisco.com/thread/39098 )
