CsFirewall Cookbook
===================
This cookbook enforces the firewall rules in cloudstack via a firewall 
management node that interacts with the API

Make sure you assign at least one node in the network the CsFirewall::manage 
role if you want to have your rules enforce in cloudstack. These machines are 
the machines that will actually talk to the Cloud Stack API via http(s)

Requirements
------------
The cloudstack_helper gem needs to be installed on the firewall management
node(s) to access the API
If you use embeeded Ruby, make sure that you install it in this version

Attributes
----------
TODO: List you cookbook attributes here.

e.g.
#### CsFirewall::default
<table>
  <tr>
    <th>Key</th>
    <th>Type</th>
    <th>Description</th>
    <th>Default</th>
  </tr>
  <tr>
    <td><tt>['cloudstack']['url']</tt></td>
    <td>String</td>
    <td>The cloudstack API url. Only needed on the manager node.</td>
    <td><tt></tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['APIkey']</tt></td>
    <td>String</td>
    <td>The cloudstack API key. Only needed on the manager node.</td>
    <td><tt></tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['SECkey']</tt></td>
    <td>String</td>
    <td>The cloudstack Secret key. Only needed on the manager node.</td>
    <td><tt></tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']</tt></td>
    <td>Object</td>
    <td>Contains firewall config</td>
    <td><tt></tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['unmanaged']</tt></td>
    <td>Object</td>
    <td>If set to true, this node will not be considered when enumerating 
        firewall rules
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['fwcleanup']</tt></td>
    <td>Boolean</td>
    <td>
	Should firewall rules not matching node attributes be cleaned up?
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['forwardcleanup']</tt></td>
    <td>Boolean</td>
    <td>
	Should port forward rules not matching node attributes be cleaned up?
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['egresscleanup']</tt></td>
    <td>Boolean</td>
    <td>
	Should egress rules not matching node attributes be cleaned up?
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['aclcleanup']</tt></td>
    <td>Boolean</td>
    <td>
	Should acl rules not matching node attributes be cleaned up?
	ACLs are only cleaned up when at least one node has specified an ACL in the network
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['maxdelete']</tt></td>
    <td>Integer</td>
    <td>
        This is the maximum number of rules CsFirewall is allowed to delete of a single type in a single run
        A negative value disables this check (*CAUTION!*)
    </td>
    <td><tt>5</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['cleanup']</tt></td>
    <td>Boolean</td>
    <td>
        Global cleanup on switch. If this attribute is set to true, any type of rule will be cleaned up.
        If this is set to false, but a more specific clean attribute is set to true, cleanup of that specific type WILL happen
    </td>
    <td><tt>False</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['scope']</tt></td>
    <td>Array</td>
    <td>
        Global scope definition for the Chef node search. Will be joined with AND, example: ['chef_environment:prod', 'hostname:web'] will only manage hosts named 'web' in the 'prod' environment.
    </td>
    <td><tt>Empty</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['ingress'][&lt;tag&gt;]</tt></td>
    <td>Array</td>
    <td>Note, use a unique tag per role to prevent roles overwriting each other
    This array holds the actual firewall and portnat rules
    Each of the rules is specified in the following format:
    <ul>
      <li>IP address
      <li>Protocol (tcp|udp|icmp)
      <li>CIDR block
      <li>Start port public|icmp type
      <li>End port public|icmp code
      <li>Start port private
    </ul>
    E.g. to specify that external TCP port 80 and 81 on ip 1.2.3.4 have to be allowed publicly and forwarded to port 8080 and 8081 specify:<br>
    [ [ "1.2.3.4", "tcp", "0.0.0.0/0", "80", "81", "8080" ] ]<br>
    </td>
    <td><tt>Empty</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['egress'][&lt;tag&gt;]</tt></td>
    <td>Array</td>
    <td>Note, use a unique tag per role to prevent roles overwriting each other
    This array holds the actual firewall and portnat rules
    Each of the rules is specified in the following format:
    <ul>
      <td>Network
      <td>CIDR block
      <td>Protocol (tcp|udp|icmp)
      <td>Start port|icmp type
      <td>End port|icmp code
    </ul>
		Notes:
    <ul>
			<li>The keyword nic_# will be replaced with the network the machine is in if nic_# is found in the network field
    </ul>
    E.g. to specify that tcp and udp port 53 traffic and all icmp traffic should be allowed out
    [<br> 
    &nbsp;&nbsp;[ "nic_0", "0.0.0.0/0", "tcp", "53", "53" ],<br>
    &nbsp;&nbsp;[ "nic_0", "0.0.0.0/0", "udp", "53", "53" ],<br>
    &nbsp;&nbsp;[ "nic_0", "0.0.0.0/0", "imcp", "-1", "-1" ],<br>
    ]<br>
    </td>
    <td><tt>Empty</tt></td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['firewall']['iptables'][&lt;chain&gt;]</tt></td>
    <td>string</td>
    <td>
      These attributes control the default policies for the &lt;chain&gt;
      To set a default blocking policy on all default chains: <br>
      { "cloudstack" : { <br>
      &nbsp;&nbsp; "firewall" : { <br>
      &nbsp;&nbsp;&nbsp;&nbsp; "iptables" : { <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "INPUT" : "DROP", <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "FORWARD" : "DROP", <br>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "OUTPUT" : "DROP" <br>
      &nbsp;&nbsp;&nbsp;&nbsp;   } <br>
      &nbsp;&nbsp; } <br>
      }
    </td>
    <td>The following rules have default values:
      <ul>
        <li>INPUT - ACCEPT
        <li>FORWARD - DROP
        <li>OUTPUT - ACCEPT
      </ul>
    </td>
  </tr>
  <tr>
    <td><tt>['cloudstack']['acl'][&lt;tag&gt;]</tt></td>
    <td>Array</td>
    <td>Use a unique tag to prevent roles from overwriting firewall rules from other roles
    This array holds the actual network ACL rules for this node
    Each of the rules is specified in the following format:
    <ul>
      <li>Network name (or nic_#)
      <li>CIDR block (may contain nic_#)
      <li>Protocol (tcp|udp|icmp)
      <li>Start port or icmp type
      <li>End port or icmp code
      <li>Direction (Ingress|Egress) *Mind the capital*, may be command sparated<td>
    </ul>
    <ul>
	<li>The keyword nic_# will be replaced with the network the machine is in if nic_# is found in the network field
        <li>Node searches can be specified by using curly braches ({}), e.g. {role:domain_controller}, will expand to a list of chef IP addresses of machines with the role domain controller
    </ul>
    E.g. to specify that on network XXX_p_FRONT
    <ul>
      <li>192.168.98.64/26 and 192.168.99.64/26 should be allowed in on tcp port 666 and 667
      <li>all ICMP on the network nic_0 is allowed in
      <li>allow tcp and udp 53 from this network to the dns server 
      <li>Allow outgoing mysql connections to the db servers
    </ul>
    Specify:<br>
    [ <br>
		&nbsp;&nbsp;	[ "XXX_p_FRONT", "192.168.98.64/26,192.168.99.64/26", "tcp", "666", "667", "Ingress" ], <br>
    &nbsp;&nbsp;  [ "nic_0", "192.168.98.64/26,192.168.99.64/26", "tcp", "666", "667", "Ingress" ], <br>
    &nbsp;&nbsp;  [ "nic_0", "{role:dnsserver}", "tcp", "53", "53", "Ingress,Egress" ], <br>
    &nbsp;&nbsp;  [ "nic_0", "{role:dnsserver}", "udp", "53", "53", "Ingress,Egress" ], <br>
    &nbsp;&nbsp;  [ "nic_0", "{role:dbserver}", "tcp", "3306", "3306", "Egress" ] <br>
		]<br>
    </td>
    <td><tt>Empty</tt></td>
  </tr>
</table>

Usage
-----
#### CsFirewall::default
This recipe does nothing, but tells the Firewall manager to read this hosts attributes for firewall input

Just include `CsFirewall` in your node its `run_list`:

```json
{
  "name":"my_node",
  "run_list": [
    "recipe[CsFirewall]"
  ]
}
```

And add rules to the normal attributes:
```json
{
  "cloudstack" : {
    "firewall" :{
      "ingress" : {
        "webserver" : [
          [ "1.2.3.4", "tcp", "0.0.0.0/0", "80", "81", "8080" ]
        ]
      },
      "egress" : {
        "dnsclient" : [
          [ "nic_0", "8.8.8.8/32", "tcp", "53", "53" ],
          [ "nic_0", "8.8.8.8/32", "udp", "53", "53" ]
        ]
      }
    },
    "acl" : {
      "appserver" : [
        [ "XXX_p_FRONT", "192.168.98.64/26,192.168.99.64/26", "tcp", "666", "667", "Ingress" ],
        [ "nic_0", "192.168.98.64/26,192.168.99.64/26", "tcp", "666", "667", "Ingress" ],
        [ "nic_0", "{role:dnsserver}", "tcp", "53", "53", "Egress" ],
        [ "nic_0", "{role:dnsserver}", "udp", "53", "53", "Egress" ]
      ]
    }
  }
}
```

#### CsFirewall::manager
This recipe tells the node to manage the Cloud Stack firewall

Just include `CsFirewall::manager` in your node its `run_list`:

```json
{
  "name":"my_node",
  "run_list": [
    "recipe[CsFirewall::manager]"
  ]
}
```

And add configuration attributes
```json
{
  "cloudstack" : {
    "url" : "https://.../client/api",
    "APIkey" : "qmFEFfAr3q-...",
    "SECkey" : "ZOAXv1WLXRfFvxD-..",
    "firewall" :{
      "cleanup" : true
    }
  }
}
```

#### CsFirewall::hostbased
This recipe tell the node to take its cloudstack firewall rules and create and maintain 
hostbased firewall rules from them.

Depending of the OS flavour, this recipe will call an addition recipe

* Linux -> CsFirewall::hostbased
* Other -> Not yet implemented

#### CsFirewall::hostbased
This recipe tell the node to take its cloudstack firewall rules and create and maintain 
hostbased firewall rules from them.

This recipe is intended to be called via CsFirewall::hostbased

If you want the default policy for the firewall altered you need to overwrite the following 
default properties:

```json
{
  "cloudstack" : {
    "firewall" : {
      "iptables" : {
        "INPUT" : "DROP",
        "OUTPUT" : "DROP",
        "FORWARD" : "DROP"
      }
    }
  }
{
```


Contributing
------------
1. Fork the repository on https://www.github.com/schubergphilis/CsFirewall
2. Create a named feature branch (like `add_component_x`)
3. Write you change
4. Write tests for your change (if applicable)
5. Run the tests, ensuring they all pass
6. Submit a Pull Request using Github

License and Authors
-------------------
Authors: 
* Frank Breedijk <fbreedijk@schubergphilis.com>
* Thijs Houtenbos <thoutenbos@schubergphilis.com>
