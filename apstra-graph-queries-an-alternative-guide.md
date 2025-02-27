# Apstra Graph Queries - An Alternative Guide<br>

**Author:** Curtis Call<br>
**Version:** 1.04<br>
**Last Modified:** February 27, 2025<br>

# Purpose

This guide was written to make Apstra graph queries more accessible to Apstra users, particularly non-developers. Whether you need to craft a graph query for a custom analytics probe or just want to explore the graph database via Apstra's Graph Explorer, the following guide will help you understand how to do this and all the different options available to you.

While this guide was written to be understandable for non-developers, some basic familiarity with programming is beneficial, such as the role of functions and variables and their common syntax. In addition, while developers are not the target audience, they will likely still benefit from the documentation of the functions and other insight into graph queries provided below.

All information is based on Apstra 5.1.0.

# Graph Databases Overview

Apstra uses a graph database to store the intended design of its datacenter fabric blueprints. A graph database has two main types of data: nodes and relationships. Nodes, sometimes called vertices, are units of data, such as systems, interfaces, routing zones, etc. Nodes are interconnected via relationships, sometimes called edges. In Apstra's graph database, these relationships are unidirectional: they have one source node and one destination node.

It's important to understand that there is no hierarchy in the graph database. There is no root node, no parents, no children. There are just nodes, interconnected to each other via relationships in various ways. And these relationships between the nodes allow one to start with one node and then quickly jump across to other related nodes in order to pull the desired information from the database.

Here is an example of a node of type "system", which stores data for a switch or generic system. In this example, we are looking at the data for a spine switch:

```json
{
  "system_node": {
    "id": "kzGyd1FQgHxw9y18uA",
    "type": "system",
    "label": "spine1",
    "access_l3_peer_link_port_channel_id_max": null,
    "access_l3_peer_link_port_channel_id_min": null,
    "deploy_mode": "deploy",
    "external": false,
    "group_label": null,
    "hostname": "spine1",
    "management_level": "full_control",
    "port_channel_id_max": null,
    "port_channel_id_min": null,
    "position_data": {
      "plane": 0,
      "pod": 0,
      "position": 0,
      "region": 0,
      "rg_position": null
    },
    "property_set": null,
    "role": "spine",
    "system_id": "5254001FA979",
    "system_index": 1,
    "system_type": "switch",
    "tags": null
  }
}
```

Relationships interconnect nodes. One type of relationship that system nodes have is `hosted_interfaces`. Here is an example of the data for that relationship:

```json
{
  "hosted_interface": {
    "id": "dVrlKhmS4V282simUA",
    "type": "hosted_interfaces",
    "property_set": null,
    "source_id": "kzGyd1FQgHxw9y18uA",
    "tags": null,
    "target_id": "RfEhu-Rscl82v3CAdw"
  }
}
```

As you can see, it has a `source_id` and a `target_id`, which refer to the `id` values of the source node and destination nodes, respectively.

## Database Schema

The graph database has a schema, which defines the types of properties each node has, their valid values, etc.

You can view the schema within the Graph Explorer. To access the Graph Explorer within the Apstra UI, click on the Platform menu item and then select Developers. Next, scroll down and select the GraphQE Explorer option. This takes you to the Graph Explorer screen. From here, you can click on the "Show reference design schema" button, and you will see a UI representation of the schema information. Click on the type of node you are interested in and an information box will pop up, showing its properties. At the top right of the box, you will see a document icon. Click on it and you will be shown a JSON representation of the schema for the node type.

For example, here is the schema for the rack node type:

```json
{
  "type": "object",
  "properties": {
    "label": {
      "type": "string",
      "default": null,
      "minLength": 1
    },
    "ip_version": {
      "type": "string",
      "default": "ipv4",
      "description": "DEPRECATED. IP version to use for server<->leaf links addresses. This option is used only for L3 servers.",
      "enum": [
        "ipv4",
        "ipv6",
        "ipv4_ipv6"
      ]
    },
    "position": {
      "type": "integer",
      "default": 1
    },
    "property_set": {
      "type": "object",
      "additionalProperties": {
        "type": "string"
      },
      "default": null
    },
    "rack_type_json": {
      "type": "string",
      "default": null,
      "minLength": 1
    },
    "tags": {
      "type": "array",
      "default": null,
      "items": {
        "type": "string"
      }
    }
  },
  "title": "rack"
}
```

## Apstra Graph Databases

Within the Graph Explorer you'll notice there is a "Type:" option in the upper right that lets you choose between 4 different graph databases for your query: config, staging, deployed, and operation.

The short explanation is if you want to query the staging configuration, then use the staging database, and if you want to query the active configuration, then use the operation database, and you can ignore the other two databases. (Analytics probe graph queries always use the operation database)

The long explanation is that the config database is the basic user input for the staged configuration, but it hasn't been expanded/enriched into the full configuration needed for deployment. For example, if you look at interface nodes in the config database, you'll see they have no `if_name` assigned, whereas in the staging database they do. And the operation database is the same as the deployed database, except it also includes dynamic intent provided from outside of Apstra, for example from VMWare integration. Bottom line: You'll likely always want to query either the staging database or the operation database.

# Graph Query Overview

The goal of a graph query is to extract nodes and/or relationships from the graph database for analysis via the Graph Explorer or for use by an analytics probe.

To understand graph queries, let's first look at the data they return. This is the type of output you can expect to see from a query within the Graph Explorer:

```json
{
  "count": 3,
  "items": [
    {
      "policy_node": {
        "id": "XETLlDL7To5avFh0rQ",
        "type": "policy",
        "label": null,
        "description": null,
        "dhcp_servers": [
          "198.51.100.2",
          "fc01:a05:198:51:100::2"
        ],
        "enabled": true,
        "policy_type": "dhcp_relay",
        "property_set": null,
        "tags": null
      }
    },
    {
      "policy_node": {
        "id": "tlM_3QNkzZebnsvOoA",
        "type": "policy",
        "label": null,
        "description": null,
        "dhcp_servers": [
          "198.51.100.2",
          "fc01:a05:198:51:100::2"
        ],
        "enabled": true,
        "policy_type": "dhcp_relay",
        "property_set": null,
        "tags": null
      }
    },
    {
      "policy_node": {
        "id": "OONt10daUhrFRl83wQ",
        "type": "policy",
        "label": null,
        "description": null,
        "dhcp_servers": [
          "198.51.100.2",
          "fc01:a05:198:51:100::2"
        ],
        "enabled": true,
        "policy_type": "dhcp_relay",
        "property_set": null,
        "tags": null
      }
    }
  ]
}
```

You can see here a JSON object with a couple fields: `count`, which indicates how many items were returned, and `items`, which is a list of the different items (paths of nodes/relationships) returned by the query.

The only structure you need to actually care about is the list of objects within the `items` list. This is the data that Apstra will be working with when pulling data via a graph query for an analytics probe, for example.

## Graph Query Chain

A graph query is composed as a chain of one or more steps, with each step performed by a function, as the graph database is traversed from node, to relationship, to node, etc.

Function calls are written similar to C, Python, etc with the function name followed by opening and closing parenthesis. Function arguments are included within the parenthesis, with commas separating them if there is more than one argument. Most arguments are in the form `variable_name=value` but just the value can be specified in certain circumstances (mentioned below).

Functions can be chained together, in which case a period separates them. Query chains are executed sequentially from left to right.

White space (newlines, spaces) can be added to make the query easier to read.

Here is an example of a query chain. Each of these functions will be described below:

`node("system", name="system_node").out().node("interface", name="interface_node")`

## The node() function

At the beginning of a graph query is the `node()` function, which selects one or more nodes from the graph database based on the arguments provided to it. Remember that there is no hierarchy within the graph database, no root node, so the first `node()` instance within a graph query is capable of selecting ANY of the nodes within the database, whereas later calls to the `node()` function within the graph query will only have access to those nodes who are connected to the prior nodes via a relationship.

To understand how graph nodes are selected, you need to understand that every graph node has a type, an id, and various properties (based on what type of node it is). A node can be selected, therefore, by referring to its type, id, or one or more of its properties, with multiple nodes selected if the match specified by the graph query applies to more than one node.

It is common to filter based on the node's type. This can be done in the `node()` function either by explicitly referring to the `type` parameter, or by indicating the type desired by placing it as the first argument to `node()`. For example, both of these function calls select all interface type nodes.

`node(type='interface')`

`node('interface')`

With the above call, all interface nodes would be selected. If specific interfaces are desired, then properties can be checked by specifying their name, an equals sign, and the value that must be matched. For example, the following query will match all interface nodes with an `if_name` property of "ge-0/0/0":

`node("interface", if_name="ge-0/0/0")`

In additional to the various properties, each node has a unique id, which can be matched in the same way as shown above by specifying `id` as the property that is being matched.

However, if you execute the above query in Graph Explorer, you will get the following output:

```json
{
  "count": 5,
  "items": [
    {},
    {},
    {},
    {},
    {}
  ]
}
```

So, five interface nodes matched the query, but zero were returned. Why? The reason is because a graph query will only return nodes that have been assigned a name, which was not done in the previous queries. To assign a name, you specify `name="NAME"`, with "NAME" being whatever you wish the nodes to be called within the query's output.

So, with this modified query:

`node('interface', name='interface_node', if_name='ge-0/0/0')`

We receive this output:

```json
{
  "count": 5,
  "items": [
    {
      "interface_node": {
        "id": "RBxGgLo3nTK1jqUu-Q",
        "type": "interface",
        "label": null,
        "description": "facing_leaf1:ge-0/0/1",
        "evpn_esi_mac": null,
        "if_name": "ge-0/0/0",
        "if_type": "ip",
        "ipv4_addr": "172.16.0.6/31",
        "ipv4_addr_type": null,
        "ipv4_enabled": null,
        "ipv6_addr": null,
        "ipv6_addr_type": null,
        "ipv6_enabled": null,
        "l3_mtu": 9170,
        "lag_mode": null,
        "loopback_id": null,
        "mlag_id": null,
        "mode": null,
        "operation_state": "up",
        "po_control_protocol": null,
        "port_channel_id": null,
        "property_set": null,
        "protocols": "ebgp",
        "ref_count": null,
        "subintf_id": null,
        "tags": null,
        "vlan_id": null
      }
    },
    {
      "interface_node": {
        "id": "RfEhu-Rscl82v3CAdw",
        "type": "interface",
        "label": null,
        "description": "facing_leaf1:ge-0/0/0",
        "evpn_esi_mac": null,
        "if_name": "ge-0/0/0",
        "if_type": "ip",
        "ipv4_addr": "172.16.0.0/31",
        "ipv4_addr_type": null,
        "ipv4_enabled": null,
        "ipv6_addr": null,
        "ipv6_addr_type": null,
        "ipv6_enabled": null,
        "l3_mtu": 9170,
        "lag_mode": null,
        "loopback_id": null,
        "mlag_id": null,
        "mode": null,
        "operation_state": "up",
        "po_control_protocol": null,
        "port_channel_id": null,
        "property_set": null,
        "protocols": "ebgp",
        "ref_count": null,
        "subintf_id": null,
        "tags": null,
        "vlan_id": null
      }
    },
    {
      "interface_node": {
        "id": "o44C5Z71tc1DUa4ZPw",
        "type": "interface",
        "label": null,
        "description": "facing_spine1:ge-0/0/1",
        "evpn_esi_mac": null,
        "if_name": "ge-0/0/0",
        "if_type": "ip",
        "ipv4_addr": "172.16.0.3/31",
        "ipv4_addr_type": null,
        "ipv4_enabled": null,
        "ipv6_addr": null,
        "ipv6_addr_type": null,
        "ipv6_enabled": null,
        "l3_mtu": 9170,
        "lag_mode": null,
        "loopback_id": null,
        "mlag_id": null,
        "mode": null,
        "operation_state": "up",
        "po_control_protocol": null,
        "port_channel_id": null,
        "property_set": null,
        "protocols": "ebgp",
        "ref_count": null,
        "subintf_id": null,
        "tags": null,
        "vlan_id": null
      }
    },
    {
      "interface_node": {
        "id": "Z4cjFDQ8xqjucn0NWQ",
        "type": "interface",
        "label": null,
        "description": "facing_spine1:ge-0/0/2",
        "evpn_esi_mac": null,
        "if_name": "ge-0/0/0",
        "if_type": "ip",
        "ipv4_addr": "172.16.0.5/31",
        "ipv4_addr_type": null,
        "ipv4_enabled": null,
        "ipv6_addr": null,
        "ipv6_addr_type": null,
        "ipv6_enabled": null,
        "l3_mtu": 9170,
        "lag_mode": null,
        "loopback_id": null,
        "mlag_id": null,
        "mode": null,
        "operation_state": "up",
        "po_control_protocol": null,
        "port_channel_id": null,
        "property_set": null,
        "protocols": "ebgp",
        "ref_count": null,
        "subintf_id": null,
        "tags": null,
        "vlan_id": null
      }
    },
    {
      "interface_node": {
        "id": "gyLhBtEFaGkpyQK9tQ",
        "type": "interface",
        "label": null,
        "description": "facing_spine1:ge-0/0/0",
        "evpn_esi_mac": null,
        "if_name": "ge-0/0/0",
        "if_type": "ip",
        "ipv4_addr": "172.16.0.1/31",
        "ipv4_addr_type": null,
        "ipv4_enabled": null,
        "ipv6_addr": null,
        "ipv6_addr_type": null,
        "ipv6_enabled": null,
        "l3_mtu": 9170,
        "lag_mode": null,
        "loopback_id": null,
        "mlag_id": null,
        "mode": null,
        "operation_state": "up",
        "po_control_protocol": null,
        "port_channel_id": null,
        "property_set": null,
        "protocols": "ebgp",
        "ref_count": null,
        "subintf_id": null,
        "tags": null,
        "vlan_id": null
      }
    }
  ]
}
```

So the query still matches the same five nodes, but now those nodes are returned within objects that are each named "interface_node".

The rules for names are the same as the rules for Python variable names, specifically: they must start with a letter or underscore; they can only contain letters, numbers, and underscores; they are case-sensitive; and they shouldn't use any of the Python keywords (listed here: [Python Keywords](https://docs.python.org/3/reference/lexical_analysis.html#keywords)).

(Note: Hyphens will (usually) technically work within node names, but using them isn't compatible with the `where()` function, and possibly in other scenarios, so it's advisable to not use them.)

It's critical to understand that `name='interface_node'` and `if_name='ge_0/0/0'` are handled very differently by the above query. The `name` argument is unique. In all other cases, arguments in that format are treated as filters that determine whether or not a node is selected. `if_name` is a property, so only interface nodes with an `if_name` property that matches "ge-0/0/0" will be selected. In contrast, the `name` argument is not a filter. It is an instruction for the graph query to use that name for the nodes it returns via that `node()` function.

(Note that you technically can provide a name value positionally within the `node()` function, the same as with the type, as the second argument of `node()`, but this will just cause confusion because `name` has a different position in the `out()` and `in_()` functions. If this statement makes no sense to you, that's fine. It isn't important. Just keep using `name="NAME"` in your `node()` syntax).

## Traversing Relationships via out() and in_()

Simply selecting a group of nodes via the `node()` function is unlikely to always provide you with the information desired. Instead, you will want to jump from node to node via their relationships. This is done through the `out()` function (to traverse outbound relationships) and the `in_()` function (to traverse inbound relationships). (And, yes, the `in_()` function includes an underscore in its name.)

Keep in mind that with `out()` and `in_()` you aren't starting from scratch, looking at the entire graph database, as you are within the first `node()` function in a query chain. Instead, you are looking at just the relationships connected to the nodes that were selected by the prior `node()` function in the chain. The same principal applies for subsequent `node()` functions, which can only look at the nodes selected by the prior `out()` or `in_()` function that preceeded them.

The rules of `out()` and `in_()` are the same as `node()`. You don't have to select a type, in which case all the relationships from the selected node will be considered, but if you do, you can match either via specifying `type="TYPE"` or else just placing the "TYPE" as the first argument. Properties can also be matched, the same as with the `node()` function, to filter which relationships you want the query to match. And a name can be provided to the relationship object if you wish it to be present in the output.

Suppose we want to match all system nodes that are connected to the "spine1" system node. We could do so with a query similar to the following, which also shows how whitespace can be added to queries to make them more readable:

```
node('system', hostname="spine1")
  .out("hosted_interfaces")
  .node("interface")
  .out("link")
  .node("link")
  .in_("link")
  .node("interface")
  .in_("hosted_interfaces")
  .node("system", name="connected_nodes", hostname=ne("spine1"))
```

Let's walk through this query chain, but first you should understand that systems have an outbound relationship to interfaces, which have an outbound relationship to links (so a link node is the connection point between the two interface nodes):

system > interface > link < interface < system

So, to start with systems, and then return to systems, we traverse outbound to the interface node and then the link node, and then start following the relationships backwards and traverse inbound relationships back to the remote systems. (The relationship between system nodes and interface nodes is of type hosted_interfaces. The relationship between interface nodes and link nodes is of type link.)

Let's walk though the query steps:

1. `node('system', hostname="spine1")` - Select the system node with hostname "spine1"
2. `out("hosted_interfaces")` - Select all outbound hosted_interface relationships off of the previously selected system node in the prior step
3. `node("interface")` - Select all interface nodes off of the relationships selected in the prior step
4. `out("link")` - Select all outbound link relationships off of the previously selected interface nodes in the prior step
5. `node("link")` - Select all link nodes connected via the relationships selected in the prior step
6. `.in_("link")` - Select all inbound link relationships to the link nodes selected in the prior step
7. `node("interface")` - Select all interface nodes connected via the relationships selected in the prior step
8. `in_("hosted_interfaces")` - Select all inbound hosted_interface relationships to the interface nodes selected in the prior step
9. `node("system", name="connected_nodes", hostname=ne("spine1"))` - Select all system nodes connected via the relationship selected in the prior step if their hostname is not `'spine1'`. The `ne()` function is a property matching function, which will be described below. We need to make sure we don't accidentally match the "spine1" node, which our forwards-and-then-backwards traversal through the graph database would otherwise have done.

Note that you don't have to actually specify the relationship type in most cases, since the node types on each side of the `out()` or `in_()` will usually  end up limiting the results to the desired type by default. So the following provides the same final output:

```
node('system', hostname="spine1")
  .out()
  .node("interface")
  .out()
  .node("link")
  .in_()
  .node("interface")
  .in_()
  .node("system", name="connected_nodes", hostname=ne("spine1"))
```

Notice that a name was only specified in the last function of the chain. This is because we only care about those nodes, so we only need to receive them within our results, which look like this:

```json
{
  "count": 3,
  "items": [
    {
      "connected_nodes": {
        "id": "-LLiXFXYXTdDJDDw6Q",
        "type": "system",
        "label": "leaf1",
        "access_l3_peer_link_port_channel_id_max": null,
        "access_l3_peer_link_port_channel_id_min": null,
        "deploy_mode": "deploy",
        "external": false,
        "group_label": "evpn-esi",
        "hostname": "leaf1",
        "management_level": "full_control",
        "port_channel_id_max": null,
        "port_channel_id_min": null,
        "position_data": {
          "plane": null,
          "pod": null,
          "position": null,
          "region": null,
          "rg_position": 0
        },
        "property_set": null,
        "role": "leaf",
        "system_id": "52540088D432",
        "system_index": 3,
        "system_type": "switch",
        "tags": null
      }
    },
    {
      "connected_nodes": {
        "id": "vdMTITQzcOdJtkthhQ",
        "type": "system",
        "label": "leaf2",
        "access_l3_peer_link_port_channel_id_max": null,
        "access_l3_peer_link_port_channel_id_min": null,
        "deploy_mode": "deploy",
        "external": false,
        "group_label": "evpn-esi",
        "hostname": "leaf2",
        "management_level": "full_control",
        "port_channel_id_max": null,
        "port_channel_id_min": null,
        "position_data": {
          "plane": null,
          "pod": null,
          "position": null,
          "region": null,
          "rg_position": 1
        },
        "property_set": null,
        "role": "leaf",
        "system_id": "5254009AA5A5",
        "system_index": 4,
        "system_type": "switch",
        "tags": null
      }
    },
    {
      "connected_nodes": {
        "id": "GVzsonAFrcRqgUdjLw",
        "type": "system",
        "label": "leaf3",
        "access_l3_peer_link_port_channel_id_max": null,
        "access_l3_peer_link_port_channel_id_min": null,
        "deploy_mode": "deploy",
        "external": false,
        "group_label": "evpn-single",
        "hostname": "leaf3",
        "management_level": "full_control",
        "port_channel_id_max": null,
        "port_channel_id_min": null,
        "position_data": null,
        "property_set": null,
        "role": "leaf",
        "system_id": "5254003F971A",
        "system_index": 5,
        "system_type": "switch",
        "tags": null
      }
    }
  ]
}
```
## Matching Multiple Query Chains

While the `node()` function will work in most cases, you might have situations where you wish to combine the results of multiple query chains. This can be accomplished by using the `match()` function, which accepts multiple query chains as arguments.

**Example:**

Match the system connected via "ge-0/0/1" on spine1 and the system connected via "ge-0/0/2" on spine2:

```
match(
  node("system", hostname="spine1").out().node("interface", if_name="ge-0/0/1").out().node("link").in_().node("interface").in_().node("system", name="spine1_connected", hostname=ne("spine1")),
  node("system", hostname="spine2").out().node("interface", if_name="ge-0/0/2").out().node("link").in_().node("interface").in_().node("system", name="spine2_connected", hostname=ne("spine2"))
)`
```

And the full output:

```json
{
  "count": 1,
  "items": [
    {
      "spine1_connected": {
        "id": "vdMTITQzcOdJtkthhQ",
        "type": "system",
        "label": "leaf2",
        "access_l3_peer_link_port_channel_id_max": null,
        "access_l3_peer_link_port_channel_id_min": null,
        "deploy_mode": "deploy",
        "external": false,
        "group_label": "evpn-esi",
        "hostname": "leaf2",
        "management_level": "full_control",
        "port_channel_id_max": null,
        "port_channel_id_min": null,
        "position_data": {
          "plane": null,
          "pod": null,
          "position": null,
          "region": null,
          "rg_position": 1
        },
        "property_set": null,
        "role": "leaf",
        "system_id": "5254009AA5A5",
        "system_index": 4,
        "system_type": "switch",
        "tags": null
      },
      "spine2_connected": {
        "id": "GVzsonAFrcRqgUdjLw",
        "type": "system",
        "label": "leaf3",
        "access_l3_peer_link_port_channel_id_max": null,
        "access_l3_peer_link_port_channel_id_min": null,
        "deploy_mode": "deploy",
        "external": false,
        "group_label": "evpn-single",
        "hostname": "leaf3",
        "management_level": "full_control",
        "port_channel_id_max": null,
        "port_channel_id_min": null,
        "position_data": null,
        "property_set": null,
        "role": "leaf",
        "system_id": "5254003F971A",
        "system_index": 5,
        "system_type": "switch",
        "tags": null
      }
    }
  ]
}
```

The prior use case just combined queries for two different named nodes, but a better use case for `match()` is having one query that matches nodes based on one type of relationship and then a second query that traverses the graph in a different way to return data of interest via different relationships. This works because `match()` consolidates the different results based on the node names, so if a particular named node didn't match one query chain, it won't be returned via the second one either (if the second query chain matches the same type of node with the same name).

**Example:**

Match on systems that are connected to external systems. Separately, return the device profile that has been assigned to those systems.

```
match(
  node("system", deploy_mode="deploy", external=False, name="monitored_system")
    .out()
    .node("interface")
    .out()
    .node("link")
    .in_()
    .node("interface")
    .in_()
    .node("system", external=True),
  node("system", name="monitored_system")
    .out("interface_map")
    .node("interface_map")
    .out("device_profile")
    .node("device_profile", name="profile")
)
```

The second `node()` statement above, if executed alone, would return all of the systems and their attached profiles, but because both query paths refer to system nodes with the same name, the only query results returned will be those paths of the system nodes that match both query chains.

# Data Value Syntax

You've already seen multiple examples of data values and how they are expressed in the syntax, which should be familiar to those used to Python or similar languages.

Strings are enclosed within quotation marks, either a pair of single quotes or double quotes.

**Example:**

`node(type='metadata', name="metadata_node")`

Booleans are either `True` or `False`, using that exact capitalization. No quotation marks are used.

**Example:**

`node('system', name='system', external=True)`

Lists (i.e. arrays) are enclosed within brackets `[]`, with a comma between elements. (In the reference design schema, this data type is called an array.)

`node("ip_endpoint", name="endpoint_node", tags=["blue-green"])`

(Note: `tags` is a deprecated property, as described below.)

Integers are written in numeric value, without quotes.

**Example:**

`node('rack', name='rack', position=1)`

Dictionaries (i.e. key/value mappings) are enclosed within braces `{}`, with a key string and its value separated by a colon. Multiple entries are separated by a comma. (In the reference design schema, this data type is called an object.)

`node('device_profile', name='dp', hardware_capabilities=has_items({"cpu": "x86", "form_factor": "1RU"}))`

# Property Matching Functions

While the exact matching provided via `property=VALUE` in the query chain functions can accomplish many possible matches, there are times when it's necessary to match everything except for that value, or match on a wildcard, or match one of multiple different values. In these scenarios, you can take advantage of the many different property matching functions available, which are described below.

## Property Matching Syntax

To apply property matching functions against properties when selecting nodes, use the equal sign between the property and the property matching function: `property=function(arguments)`.

Examples are shown below for each function.

## Comparison Functions

### eq()

Tests for equality. Equivalent to just using an equals sign `=`.

Works as expected with strings, booleans, and integers.

With lists, it requires an exact match of all values and their positions. Because of this, it would likely be better to use the `where()` function with a lamba expression to match lists instead. (An example is shown below in the section about the `where()` function.)

With dictionaries, it requires that all keys and values be present. Most likely, the `has_items()` function, described below, would work better.

**Examples:**

Match all external systems:

`node('system', name='system', external=eq(True))`

Match all interfaces with a name of "eth2":

`node("interface", name="interface", if_name=eq("eth2"))`

### aeq()

Matches strings using wildcard patterns. Only strings are supported and the match is case sensitive.

Supported wildcard patterns:

- `*` - Matches all characters
- `?` - Matches a single character
- `[seq]` - Matches a character within a specified range. For example `[a,g,f]`, which matches those three specific characters or `[a-m]`, which matches all characters from a through m.
- `[!seq]` - Matches any character that isn't in the specified range.

Note: To match a special character, enclose it within a `[]` sequence. (This doesn't work to match a closing `]`).

**Examples:**

Match all leaf1 interfaces that begin within "ge-":

`node("system", label="leaf1").out().node("interface", if_name=aeq("ge-*"), name="interface")`

Match systems leaf1 and leaf2 (2 options are shown):

`node("system", name="system", label=aeq("leaf[1,2]"))`

`node("system", name="system", label=aeq("leaf[1-2]"))`

### ne()

Tests for inequality. The opposite of the `eq()` function.

Works as expected with strings, booleans, and integers.

As with the `eq()` function, it is not recommended for lists or dictionaries.

**Examples:**

Match all interfaces that aren't named "ge-0/0/0":

`node("interface", name="interface", if_name=ne("ge-0/0/0"))`

Match all racks that aren't in position 1:

`node("rack", name="rack", position=ne(1))`

### gt()

Tests if the property is greater than the specified value.

Works as expected with integers. With strings it compares them lexicographically (i.e. alphabetically).

If a node's property value is null (None), it will not match.

Not recommended for booleans, lists, or dictionaries.

**Examples:**

Match all racks that are greater than position 1:

`node("rack", name="rack", position=gt(1))`

Match all interfaces whose names are lexicographically greater than "ge-0/0/0" (e.g. "ge-0/0/2", "lo0.0", etc):

`node("interface", name="interface", if_name=gt("ge-0/0/0"))`

### ge()

Tests if the property is greater than or equal to the specified value.

Works as expected with integers. With strings it compares them lexographically (i.e. alphabetically).

Note: As of Apstra 5.1.0 this function incorrectly matches if the property value is null. To workaround this behavior, you can combine it with the `_and()` and `not_none()` functions (both are described below): `_and( ge(VALUE), not_none() )`.

Not recommended for booleans, lists, or dictionaries.

**Examples:**

Match all virtual networks whose L3 MTU is greater than or equal to 9000:

`node('virtual_network', name='vn', l3_mtu=ge(9000))`

### lt()

Tests if the property is less than the specified value.

Works as expected with integers. With strings it compares them lexicographically (i.e. alphabetically).

If a node's property value is null (None), it will not match.

Not recommended for booleans, lists, or dictionaries.

**Examples:**

Match all virtual networks whose VLAN ID is less than 37:

`node('virtual_network', name='vn', reserved_vlan_id=lt(37))`

### le()

Tests if the property is less than or equal to the specified value.

Works as expected with integers. With strings it compares them lexicographically (i.e. alphabetically).

Note: As of Apstra 5.1.0 this function incorrectly matches if the property value is null. To workaround this behavior, you can combine it with the `_and()` and `not_none()` functions (both are described below): `_and( le(VALUE), not_none() )`.

Not recommended for booleans, lists, or dictionaries.

**Examples:**

Match all racks whose position is less than or equal to 2:

`node("rack", name="rack", position=le(2))`

## Multiple-Option Match Functions

### is_in()

Allows you to match a string or integer property value against multiple possible options, which are provided to the `is_in()` function as a list (i.e. enclosed in []).

(Technically, you can encode the parameter as a set or tuple too, but keep it simple and just use list format.)

Works as expected to match string and integer property values.

Not recommended for boolean, list, or dictionary properties.

The `is_in()` function is one of the only property matching functions (other than `eq()`) that can be used to match a node's type, as shown in the example below.

**Examples:**

Match racks in position 1 or 2:

`node("rack", name="rack", position=is_in([1, 2]))`

Match all nodes of type system or interface:

`node(type=is_in(["system", "interface"]), name="node")`

### not_in()

The reverse of the `is_in()` function. You provide a list of values that you do not want to match against. These are encoded as a list (i.e. enclosed in []).

(Technically, you can encode the parameter as a set or tuple too, but keep it simple and just use list format.)

Works as expected to match string and integer property values.

Not recommended for boolean, list, or dictionary properties.

**Example:**

Match interfaces that are not named "ge-0/0/0" or "ge-0/0/1":

`node("interface", name="interface", if_name=not_in(["ge-0/0/0", "ge-0/0/1"]) )`

## Null Property Functions

### is_none()

Matches if the property value is null (or None). The default value for many properties is null, so this is way to check if that property has been set or not.

To verify the default value for a property, check the reference design schema (viewable via the Apstra Graph Explorer). For example, this is the schema for the virtual_network reserved_vlan_id property, which shows that its default value is null:

```
    "reserved_vlan_id": {
      "type": "integer",
      "default": null,
      "maximum": 4094,
      "minimum": 1
    }
```

**Example:**

Match virtual networks that do not have a defined VLAN ID:

`node("virtual_network", name="vn", reserved_vlan_id=is_none())`

### not_none()

The opposite of `is_none()`. Matches if the property value is not null (or None).

**Example:**

Match virtual networks that have a defined VLAN ID:

`node("virtual_network", name="vn", reserved_vlan_id=not_none())`

## Dictionary Match Functions

### has_items()

Tests if a dictionary property value has the key/value pairs provided as a dictionary parameter. All of the provided key/value pairs must match key/value pairs of the node's property in order to be a match.

(Technically, any Python dict() compatible type will be accepted as an argument, but using a dictionary is recommended.)

**Example:**

Match systems that have a position_data dictionary that includes a "position" key with a value of 0:

`node("system", name="system", position_data=has_items({"position": 0 }))`

### has_keys()

Tests if a dictionary property value has the keys provided as a list parameter. All of the provided keys must match keys within the node's property in order to be a match.

(Technically, the function accepts any of these data types as an argument: list/set/tuple/frozenset, but using a list is recommended.)

**Example:**

Match device profiles that have an "asic" key within their hardware capabilities property:

`node("device_profile", name="device", hardware_capabilities=has_keys(["asic"]))`

## Logical Operation Functions

### \_and()

Combines the results of multiple property matching functions. All of the included property matching functions must match in order for the node to match.

(Note: Yes, this function name does begin with an underscore.)

**Examples:**

Match interfaces that begin with "lo0" and don't match "lo0.3":

`node("interface", name="interface", if_name=_and(aeq("lo0*"), ne("lo0.3")))`

Match virtual networks with VLANs greater than 40 and less than 50:

`node("virtual_network", name="vn", reserved_vlan_id=_and(gt(40), lt(50)))`

### \_or()

Combines the results of multiple property matching functions. If any of the included property matching functions match, then it is treated as a match.

(Note: Yes, this function name does begin with an underscore.)

**Examples:**

Match interfaces that begin with either "lo0" or "irb":

`node("interface", name="interface", if_name=_or(aeq("lo0*"), aeq("irb*")))`

Match virtual networks with VLANs either greater than 4000 or less than 100:

`node("virtual_network", name="vn", reserved_vlan_id=_or(gt(4000), lt(100)))`

### \_not()

Reverses the provided property match.

(Note: Yes, this function name does begin with an underscore.)

**Example:**

Match all interfaces that don't begin with "lo0":

`node("interface", name="interface", if_name=_not(aeq("lo0*")))`

## Tag Matching Functions

First, a word about tags: Originally, tags were handled via a `tags` property on each node, and you can still see this deprecated `tags` property on each node within the graph database, always set to null.

More recently, tags were moved into a separate tag node, which has a relationship pointing to all the nodes that have been tagged with that particular tag.

Because of this, specific tag matching functions are provided that act the same as normal property matching functions even though they technically are doing a graph traversal to find related tag nodes.

Note that all of these tag functions are used by matching with `tag`, NOT with `tags`, which is the deprecated property.

### Nodes that still use the deprecated tags property

That said, as of Apstra 5.1.0, there are a couple node types that still use the deprecated `tags` property: `ip_endpoint` and `policy`. Because of this, the tag matching functions cannot be used to match those nodes, nor will Query Tag Filters within an analytics probe work correctly.

In these corner cases, the best workaround to match on tags is to use the `where()` function (described later) with a lamba expression, like this:

`node('ip_endpoint', name="ip_endpoint").where(lambda ip_endpoint: 'test-tag' in (ip_endpoint.tags or []))`

In all other cases, use the tag matching functions described below.

### has_all()

Matches nodes that have all of the mentioned tags, which are provided to the function as a list.

(Technically, the function accepts any of these data types as an argument: list/set/tuple/frozenset, but using a list is recommended.)

**Example:**

Match systems with both the "financial" and "customer-152" tags:

`node("system", name="system", tag=has_all(["financial", "customer-152"]))`

### has_any()

Matches nodes that have any of the mentioned tags, which are provided to the function as a list.

(Technically, the function accepts any of these data types as an argument: list/set/tuple/frozenset, but using a list is recommended.)

**Example:**

Match systems with either the "financial" or "retail" tags:

`node("system", name="system", tag=has_any(["financial", "retail"]))`

### has_none()

Matches nodes that have none of the mentioned tags, which are provided to the function as a list.

(Technically, the function accepts any of these data types as an argument: list/set/tuple/frozenset, but using a list is recommended.)

**Example:**

Match servers with neither the "financial" nor "retail" tags:

`node("system", name="system", system_type="server", tag=has_none(["financial", "retail"]))`

# Advanced Matching Functions

The following functions can be attached to the query chain to further filter the matched notes.

## where()

The `where()` function is used to execute match logic not possible via the built-in property-matching functions by using a Python lambda expression instead.

(The syntax of Python lambda expressions are outside the scope of this document. See here: [Lambdas](https://docs.python.org/3/reference/expressions.html#lambda))

The properties of nodes and relationships that have been named previously within the query chain can be referred to within the lambda expression.

The `where()` function can be placed in the query chain after a `node()`, `out()`, `in_()`, or `match()` function.

**Example:**

Match all routing-policies that have "10.0.0.0/8" as one of their aggregate prefixes:

`node("routing_policy", name="policy").where(lambda policy: "10.0.0.0/8" in policy.aggregate_prefixes)`

## having()

The `having()` function is used to filter the query results to only include nodes which also match a second subquery.

The first parameter must be the query expression. This is followed by `names`, which is assigned a list of node names that should be matched between the main query and the subquery. Note that the `name=` parameter in the subquery, rather than naming the matched name, is used to indicate which node in the main query should be used in the subquery.

For example, if the main query had `node('interface', name='inf')`, and the subquery had `node(name='inf')`, then you are instructing the subquery to be executed against the nodes named 'inf' in the main query.

Ranges can be specified if desired: `at_least` or `at_most`, indicating how many node matches are expected. An `inverse` parameter can be set to `True` as well, to flip the result based on `at_least` or `at_most`. If neither `at_least` or `at_most` is defined, then it is assumed you are looking for at least one node.

The `having()` function can be placed in the query chain after a `node()`, `out()`, `in_()`, or `match()` function.

**Example:**

Match all systems that connect to the system "spine1":

```
node("system", name="system", hostname=ne("spine1"))
  .having(
    node(name="system")
    .out()
    .node("interface")
    .out()
    .node("link")
    .in_()
    .node("interface")
    .in_()
    .node("system", hostname="spine1")
    , names=["system"]
  )
```

(An alternate approach is shown for the `ensure_different()` function.)

## ensure_different()

The `ensure_different()` function is used to verify that two nodes that have been assigned different names are not actually the same graph node. It catches cases where a path in the graph database has doubled back upon itself.

It takes two parameters: the two node names that should not both refer to the same graph node.

The `ensure_different()` function can be placed in the query chain after a `node()`, `out()`, `in_()`, or `match()` function.

**Example:**

Match all systems that connect to the system "spine1":

```
node("system", name="system", hostname=eq("spine1"))
  .out()
  .node("interface")
  .out()
  .node("link")
  .in_()
  .node("interface")
  .in_()
  .node("system", name="remote_system")
  .ensure_different("system", "remote_system")
```

(An alternate approach is shown for the `having()` function.)

## distinct()

The `distinct()` function, can only be used with the `match()` function. It verifies that only one copy of a node is returned for those nodes named in the list provided to the `distinct()` function.

**Example:**

Match all tags connected to system nodes, but only show each tag once:

`match( node("tag", name="tag").out().node("system")).distinct(["tag"])`

# Predefined Query Catalog

Graph databases provide a convenient way to move from one unit of data to another, and Apstra's query language is very rich, but working with graph databases is likely a new concept for most people, and it might take some time to fully grasp how to write such queries effectively.

The good news is that Apstra itself provides a query catalog with a couple dozen queries built in, which you can use as is, or use as a starting point and then edit to your needs.

The Predefined Query Catalog can be reached via the UI's Graph Explorer by clicking on the "Predefined Query Catalog" button. On the Query Catalog window there is an Actions column to the right, where you can click on an Execute button and launch the query in the Graph Explorer.

Predefined graph queries can also be used when editing the graph query for analytics probes. In the Graph Query field, on the far right, there is a document icon with the tooltip "Select a predefined graph query". As you might expect, by clicking on that button you will be shown a window that gives you the option to select one of the predefined queries to load into your Graph Query field, which you can then edit as desired.

# Analytic Probe Graph Queries

Apstra allows you to add custom analytics probes to your blueprints, increasing the breadth of validation that is performed of your blueprint beyond what is available with Apstra natively. There are a number of predefined probes that can be used; however, if you wish to create a fully custom probe, there are a number of built-in source processors that expect you to provide a graph query and expressions to pull required data out of that graph query.

## BGP Session Processor

As a first example, let's consider the BGP Session Processor. To understand what graph query input is required for this processor, look for the red asterisks next to the fields, which indicate those fields are required. For the BGP Session Processors, these are "Graph Query" and "System ID" (under Telemetry)

The data the processor requires from the graph query is explained in the information field below the Graph Query field. In the case of the BGP Session processor, it is looking for the systems that you wish to monitor. Your query, therefore, must match only those systems you want to monitor, and it must also assign a name to those selected nodes so you can instruct Apstra on how to extract the desired data from the query results.

This can be accomplished via a single `node()` function call. For example, suppose we want to monitor the BGP sessions for all the spine switches. We could use the following graph query:

```
node("system", deploy_mode="deploy", role="spine", name="spine_switch")
```

You'll notice that as you enter this into the UI, Apstra will highlight to the left the type of nodes your returned data will consist of as well as the name you assigned to it.

Next, scroll down into the Telemetry section to find the System ID field that is required. This is where you indicate to Apstra how to extract the required system_id from the output, which it requires in order to add entries for each monitored system into the probe's data structure. Note that there is a Suggestion field below, which you can just click on if it looks correct and have the System ID field autopopulated. In this case, the correct expression would be "spine_switch.system_id". In other words NAME_OF_NODE.PROPERTY_NAME (the name of the node, followed by a period, followed by the property name for that node).

This is all that is required in order to successfully configure a BGP Session Processor. After creating the probe, you can now see data gathered for all the BGP sessions for the spine switches.

## Environment Processor

A more complicated query is required for the Environment processor because it requires a Device Profile Label, referring to the device profile assigned to the system, in addition to the System ID.

To meet this need, we can make use of the `match()` function so that one of the query chains can be used to match the systems we are interested in, and the second query chain can be used to select the device profiles for those selected systems.

Suppose we wanted to use the Environment Processor to monitor all systems that are attached to external systems. We could do so via the following Graph Query:

```
match(
  node("system", deploy_mode="deploy", external=False, name="monitored_system")
    .out()
    .node("interface")
    .out()
    .node("link")
    .in_()
    .node("interface")
    .in_()
    .node("system", external=True),
  node("system", name="monitored_system")
    .out("interface_map")
    .node("interface_map")
    .out("device_profile")
    .node("device_profile", name="profile")
)
```

The Device Profile Label can now be set to `profile.label` and the System ID to `monitored_system.system_id`.

## Generic Graph Collector Processor

This processor, which is designed to retrieve any desired data from the graph database, deserves special mention because, while its graph query has the same requirements as the other processors (nodes of interest must be selected and named), it has some unique requirements due to its flexibility in the kind of data it can retrieve.

First, the Value field must contain an expression that retrieves the data value you want included in the "Value" column for the processor. This is the standard NODE_NAME.PROPERTY_NAME syntax. However, because this could match on a variety of data types, there is an additional Data Type field where you must specify whether the Value field is a Number, String, or Discrete State.

Number and String data types are self-explanatory, but Discrete State allows you to translate numeric values into more understandable strings. To make use of it, you would include entries under Value Map, with the number configured along with the string that should be displayed in its place.

If you fill just the Graph Query, Value, and Data Type fields, you are likely to get the following error message when trying to create the probe:

```
Error configuring probe: Processor generic_graph_collector: runtime configuration error: 'The following properties are not unique across output items (): ()'
```

This will appear anytime there is more than one item returned by your graph query, which will be the case the majority of the time. The issue is that Apstra needs to know what data field will uniquely identify the different items. This is entered under the Advanced section with "Static context keys". These are one or more key/value pairs that will be added as columns into the processor's results and must provide a way to uniquely identify each separate item.

**Example:**

Display the VLAN ID for all Virtual Networks that have a VLAN configured:

Graph Query: `node("virtual_network", name="vn", reserved_vlan_id=not_none())`

Value: `vn.reserved_vlan_id`

Data Type: `Number`

Advanced > Static context keys: `Name` : `vn.label`

## Query Tag Filter

An alternate way to select the nodes you want to monitor are Query Tag Filters. Instead of having a complicated graph query to select the exact nodes, you can tag those items you are interested in. Your graph query can then select all nodes of the required type and simply give them a name.

You then add one or more Tag Filter options. In each case you will select the node name (which you assigned in your graph query), decide if you want to require the tag(s) to be present (Is In) or require the tag(s) to not be present (Not In) and then select the tags you wish to match against.

The result is that only those nodes which match the Tag Filters will be selected for monitoring.

# Apstra Graph Explorer

Let's finish up by highlighting Graph Explorer, which is useful to explore the graph database but even more useful to verify the graph queries you want to use in your analytic probes.

To access the Graph Explorer: Click on Platform > Developers and then select GraphQE Explorer.

Once in Graph Explorer, at the top of the screen you'll see a few options. For Mode, leave it as QueryEngine, which is the syntax covered by this guide. Blueprint can be changed to whichever blueprint you wish to query. Type gives an option of the 4 graph databases (described earlier). Basically, if you are interested in the staging configuration, use the staging database. Otherwise leave it at the default operation database.

A "Query Builder" option is available if you want help to craft your queries, and there is a "Predefined Query Catalog" that you can look through and test with, as described previously.

In addition, you can view the schema via the "Show reference design schema" button or navigate through the entire graph database itself graphically through the "Show full blueprint" button.

Finally, to execute a graph query, simply enter the query in the Query Editor box and then click the Execute button. You'll see the results posted below in the format shown earlier in this guide. If they don't match what you want/expect, then experiment with your query until you get the results you need.

I hope this guide was helpful.