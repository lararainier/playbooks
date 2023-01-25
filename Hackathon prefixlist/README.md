# ansible-ios-prefix-list-route-map

Prefix-lists and route-maps are the main tools for network engineers to control the route advertisements at the core and edge network infrastructure, used by dynamic routing protocols like bgp.
<br>

There are defined two roles - ios-prefix-list and ios-route-map, and an example of playbook to apply these prefix-lists and route-maps on Cisco IOS devices, just populating a txt file with prefix-lists content.<br>
The network operators can easily update the prefix-lists and route-maps, without worrying about ansible and yaml syntax.
<br>


The role "ios-prefix-list" will read the "prefixlist-name.txt" file content, and create/update a prefix-list in the target device(s) defined in the playbook.
The role "ios-route-map" will apply the prefix-list in the route-map name defined.
<br>

After apply the prefix-list and associate this to the route-map, a debug command returns the prefix-list content applied and the route-map configuration output.
 
 Example:
 
```sh 
TASK [ios-prefix-list : update prefix-list prefix-list-test] ******************************************
changed: [x.x.x.x]

TASK [ios-prefix-list : show prefix-list prefix-list-test] ********************************************
ok: [x.x.x.x]

TASK [ios-prefix-list : debug] ************************************************************************
ok: [    "prefixlistout.stdout_lines": [
        [
            "ip prefix-list prefix-list-test: 2 entries", 
            "   seq 10 permit 10.0.0.0/24", 
            "   seq 20 permit 10.10.0.0/16 le 24"
        ]
    ]
}

TASK [ios-route-map : update route-map route-map-test with prefix-list prefix-list-test] **************
changed: [x.x.x.x]

TASK [ios-route-map : show route-map route-map-test] **************************************************
ok: [x.x.x.x]

TASK [ios-route-map : debug] **************************************************************************
ok: [x.x.x.x] => {
    "routemapout.stdout_lines": [
        [
            "route-map route-map-test, permit, sequence 10", 
            "  Match clauses:", 
            "    ip address prefix-lists: prefix-list-test ", 
            "  Set clauses:", 
            "  Policy routing matches: 0 packets, 0 bytes"
        ]
    ]
}

```
<br>

# Variables

You must define the variables "prefixlist_name" and "routemap_name" in the playbook.

**prefixlist_name:** "name-of-prefix-list-in-the-device-config"<br>
**routemap_name:** "name-of-route-map-in-the-device-config"<br>

Example:

**prefixlist_name:** prefix-list-test<br>
**routemap_name:** route-map-test<br>

This is to create/update a prefix-list named "prefix-list-test" with "prefix-list-test.txt" file content, and apply this prefix-list to the route-map "route-map-test".


<br>

# Running Playbooks:

```sh
$ ansible-playbook -i hosts playbook.yaml --ask-pass
```


You can create/update more than one prefix-lists and route-maps in the same device(s), just replicating the task content and changing the variables.
You need also to create the respective files "prefix-list-name.txt" with prefix-list content.

Example:

```sh
   - name: update other prefix-list "{{ prefixlist_name }}"
    vars:
      prefixlist_name: prefix-list-test-2
    import_role:
      name: ios-prefix-list

  - name: apply other prefix-list "{{ prefixlist_name }}" on other route-map "{{ routemap_name }}"
    vars:
      prefixlist_name: prefix-list-test-2
      routemap_name: route-map-test-2
    import_role:
      name: ios-route-map
 
 ```

