# NET

## University Network Project: OSPF, NAT, & ACLs

{% file src=".gitbook/assets/ntc.pkt" %}

### Project Goals

1. VLANs: Segment traffic for Faculty, Student, and Admin departments.
2. OSPF: Use multi-area OSPF for internal dynamic routing.
3. ACLs: Implement a security policy to block Student access to the Admin network.
4. NAT: Configure PAT (overload) to allow all internal devices to access the internet via a single public IP.

<figure><img src=".gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

### Network Topology

* `SW1` (Cisco 2960): Manages the internal VLANs and connects to PCs.
* `R1` (Cisco 2911): Internal router. Acts as the gateway for all VLANs (Router-on-a-Stick) and runs OSPF.
* `R2` (Cisco 2911): Edge router. Connects the internal network to the internet, runs OSPF (ABR), and performs NAT.
* `ISP` (Cisco 2911): A simulated internet router, including a loopback for `8.8.8.8`.

### IP & VLAN Plan

| **Device**   | **Role**    | **VLAN** | **IP Address**      | **Gateway**     |
| ------------ | ----------- | -------- | ------------------- | --------------- |
| PC-Faculty   | Staff LAN   | 10       | `192.168.10.10 /24` | `192.168.10.1`  |
| PC-Student   | Student LAN | 20       | `192.168.20.10 /24` | `192.168.20.1`  |
| PC-Admin     | Admin LAN   | 30       | `192.168.30.10 /24` | `192.168.30.1`  |
| R1 (G0/0.10) | Faculty GW  | 10       | `192.168.10.1`      | N/A             |
| R1 (G0/0.20) | Student GW  | 20       | `192.168.20.1`      | N/A             |
| R1 (G0/0.30) | Admin GW    | 30       | `192.168.30.1`      | N/A             |
| R1 (G0/1)    | To R2       | N/A      | `10.0.0.1 /30`      | N/A             |
| R2 (G0/0)    | To R1       | N/A      | `10.0.0.2 /30`      | N/A             |
| R2 (G0/1)    | To ISP      | N/A      | `203.0.113.1 /24`   | `203.0.113.254` |
| ISP (G0/0)   | ISP GW      | N/A      | `203.0.113.254 /24` | N/A             |
| ISP (Lo0)    | Sim Server  | N/A      | `8.8.8.8 /32`       | N/A             |

***

### Device Configurations

#### 1. `SW1` (Cisco 2960) Configuration

Cisco CLI

```
enable
configure terminal

! Set hostname
hostname SW1

! Create VLANs
vlan 10
 name Faculty
vlan 20
 name Student
vlan 30
 name Admin
exit

! Configure Access Ports for PCs
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown

! Configure Trunk Port to Router R1
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown

end
copy running-config startup-config
```

#### 2. `R1` (Internal Router) Configuration

Cisco CLI

```
enable
configure terminal

! Set hostname
hostname R1

! --- Configure Trunk Interface ---
interface GigabitEthernet0/0
 no shutdown
exit

! --- Configure Sub-interfaces (VLAN Gateways) ---
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ! Apply the security ACL
 ip access-group STUDENT_POLICY in

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit

! --- Configure WAN Link (to R2) ---
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 no shutdown

! --- Configure Security ACL ---
ip access-list extended STUDENT_POLICY
 ! 1. Deny Students (20) from talking to Admin (30)
 deny   ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
 ! 2. Allow all other traffic
 permit ip any any

! --- Configure OSPF ---
router ospf 1
 router-id 1.1.1.1
 ! Advertise internal LANs into Area 1
 network 192.168.10.0 0.0.0.255 area 1
 network 192.168.20.0 0.0.0.255 area 1
 network 192.168.30.0 0.0.0.255 area 1
 ! Advertise WAN link into Area 0 (Backbone)
 network 10.0.0.0 0.0.0.3 area 0

end
copy running-config startup-config
```

#### 3. `R2` (Edge/Internet Router) Configuration

Cisco CLI

```
enable
configure terminal

! Set hostname
hostname R2

! --- Configure Inside Interface (to R1) ---
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 ! Define this as the "inside" for NAT
 ip nat inside
 no shutdown

! --- Configure Outside Interface (to Internet) ---
interface GigabitEthernet0/1
 ip address 203.0.113.1 255.255.255.0
 ! Define this as the "outside" for NAT
 ip nat outside
 no shutdown

! --- Configure Static Default Route (to Internet) ---
ip route 0.0.0.0 0.0.0.0 203.0.113.254

! --- Configure NAT ---
! 1. Create ACL to identify internal IPs to translate
ip access-list standard NAT_ACL
 permit 192.168.10.0 0.0.0.255
 permit 192.168.20.0 0.0.0.255
 permit 192.168.30.0 0.0.0.255

! 2. Enable NAT (PAT)
ip nat inside source list NAT_ACL interface GigabitEthernet0/1 overload

! --- Configure OSPF ---
router ospf 1
 router-id 2.2.2.2
 ! Advertise WAN link into Area 0
 network 10.0.0.0 0.0.0.3 area 0
 ! Advertise the default route to R1
 default-information originate

end
copy running-config startup-config
```

#### 4. `ISP` (Simulated Internet) Router Configuration

Cisco CLI

```
enable
configure terminal

! Set hostname
hostname ISP

! --- Configure Interface (to R2) ---
! This is the gateway R2's static route is pointing to
interface GigabitEthernet0/0
 ip address 203.0.113.254 255.255.255.0
 no shutdown

! --- Configure Loopback (to simulate 8.8.8.8) ---
interface Loopback0
 ip address 8.8.8.8 255.255.255.255
 no shutdown

end
copy running-config startup-config
```

#### 5. PC IP Configuration

* `PC-Faculty` (VLAN 10):
  * IP: `192.168.10.10`
  * Mask: `255.255.255.0`
  * Gateway: `192.168.10.1`
* `PC-Student` (VLAN 20):
  * IP: `192.168.20.10`
  * Mask: `255.255.255.0`
  * Gateway: `192.168.20.1`
* `PC-Admin` (VLAN 30):
  * IP: `192.168.30.10`
  * Mask: `255.255.255.0`
  * Gateway: `192.168.30.1`

***

### Full Verification Plan

#### Step 1: Verify Layer 2 (Switch)

On `SW1`:

Cisco CLI

```
show vlan brief
! Look for: VLANs 10, 20, 30 are "active"
! Look for: Fa0/1 in VLAN 10, Fa0/2 in 20, Fa0/3 in 30

show interfaces trunk
! Look for: G0/1 status is "trunking"
```

#### Step 2: Verify Layer 3 (Gateways)

On `PC-Faculty`:

Bash

```
ping 192.168.10.1
! Look for: "Reply from 192.168.10.1..."
```

On `PC-Student`:

Bash

```
ping 192.168.20.1
! Look for: "Reply from 192.168.20.1..."
```

#### Step 3: Verify OSPF (Dynamic Routing)

On `R1`:

Cisco CLI

```
show ip ospf neighbor
! Look for: Neighbor ID 2.2.2.2 (R2) in "FULL" state

show ip route
! Look for: "O*E2 0.0.0.0/0 [110/1] via 10.0.0.2"
! Look for: "Gateway of last resort is 10.0.0.2..."
```

On `R2`:

Cisco CLI

```
show ip ospf neighbor
! Look for: Neighbor ID 1.1.1.1 (R1) in "FULL" state

show ip route
! Look for: "O IA" routes for 192.168.10.0, 192.168.20.0, and 192.168.30.0
```

#### Step 4: Verify ACL (Security Policy)

On `PC-Student` (This ping MUST fail):

Bash

```
ping 192.168.30.10
! Look for: "Request timed out."
```

On `PC-Faculty` (This ping MUST succeed):

Bash

```
ping 192.168.30.10
! Look for: "Reply from 192.168.30.10..."
```

On `R1`:

Cisco CLI

```
show access-lists STUDENT_POLICY
! Look for: "matches" on the "deny" line (from the failed ping)
```

#### Step 5: Verify NAT (Internet Access)

On `PC-Student` (This ping MUST succeed):

Bash

```
ping 8.8.8.8
! Look for: "Reply from 8.8.8.8..."
```

On `R2` (while the ping is running):

Cisco CLI

```
show ip nat translations
! Look for: An entry translating 192.168.20.10 (Inside local)
!          to 203.0.113.1 (Inside global)
```
