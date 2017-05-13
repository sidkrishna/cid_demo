# DEMO pack to add a switch and configure IP Fabric on it

## To kick off the workflow via a webhook, run

```
curl -k -X POST https://${IP}/api/v1/webhooks/switch/add \
  -H "St2-Api-Key:${API_KEY}" \
  -H "Content-Type: application/json" \
  --data '{"host": ${SWITCH_IP}, "fabric": ${FABRIC_NAME}'
```

## Track what happened to the webhook? Did rule fire?

```
vagrant@bwctest:~/aig_demo$ st2 rule-enforcement list --rule=aig_demo.add_and_configure_ipfabric_on_switch
+--------------------------+------------------------+------------------------+------------------------+------------------------+
| id                       | rule.ref               | trigger_instance_id    | execution_id           | enforced_at            |
+--------------------------+------------------------+------------------------+------------------------+------------------------+
| 57d8968c1897127938851e87 | aig_demo.add_and_confi | 57d8968c1897127938851e | 57d8968c1897127938851e | Wed, 14 Sep 2016       |
|                          | gure_ipfabric_on_switc | 83                     | 86                     | 00:15:08 UTC           |
|                          | h                      |                        |                        |                        |
| 57d898c61897127938851f0a | aig_demo.add_and_confi | 57d898c61897127938851f | 57d898c61897127938851f | Wed, 14 Sep 2016       |
|                          | gure_ipfabric_on_switc | 06                     | 09                     | 00:24:38 UTC           |
|                          | h                      |                        |                        |                        |
+--------------------------+------------------------+------------------------+------------------------+------------------------+
vagrant@bwctest:~/aig_demo$ st2 rule-enforcement get 57d898c61897127938851f0a
+---------------------+-----------------------------------------------+
| Property            | Value                                         |
+---------------------+-----------------------------------------------+
| id                  | 57d898c61897127938851f0a                      |
| rule.ref            | aig_demo.add_and_configure_ipfabric_on_switch |
| trigger_instance_id | 57d898c61897127938851f06                      |
| execution_id        | 57d898c61897127938851f09                      |
| failure_reason      |                                               |
| enforced_at         | 2016-09-14T00:24:38.044268Z                   |
+---------------------+-----------------------------------------------+
vagrant@bwctest:~/aig_demo$
```

## Now check the incoming event

```
vagrant@bwctest:~/aig_demo$ st2 trigger-instance get 57d898c61897127938851f06
+-----------------+--------------------------------------------------------------+
| Property        | Value                                                        |
+-----------------+--------------------------------------------------------------+
| id              | 57d898c61897127938851f06                                     |
| trigger         | core.686578b5-d250-4553-be80-ac8499d75f74                    |
| occurrence_time | 2016-09-14T00:24:38.023000Z                                  |
| payload         | {                                                            |
|                 |     "body": {                                                |
|                 |         "host": "10.254.10.73",                              |
|                 |         "fabric": "new_fabric"                               |
|                 |     },                                                       |
|                 |     "headers": {                                             |
|                 |         "X-Request-Id": "d6cb1109-159e-438e-                 |
|                 | 8c89-b0606e5dd8d0",                                          |
|                 |         "X-Forwarded-For": "10.254.13.59",                   |
|                 |         "Content-Length": "48",                              |
|                 |         "Accept": "*/*",                                     |
|                 |         "User-Agent": "curl/7.35.0",                         |
|                 |         "Host": "10.254.13.59,10.254.13.59",                 |
|                 |         "St2-Api-Key": "ZDM0ZmI4MjgyYjVjZTQwMGEzYWRhY2VjZjc0 |
|                 | YWFjYjBmNDQ4NjdhYTdmMGQwNzdhYzU2ZWFlYWVjYTMwZWQ3NQ",         |
|                 |         "X-Real-Ip": "10.254.13.59",                         |
|                 |         "Content-Type": "application/json"                   |
|                 |     }                                                        |
|                 | }                                                            |
| status          | processed                                                    |
+-----------------+--------------------------------------------------------------+
vagrant@bwctest:~/aig_demo$
```

## Now check the workflow (should match UI)

```
vagrant@bwctest:~/aig_demo$ st2 execution get 57d898c61897127938851f09
id: 57d898c61897127938851f09
action.ref: aig_demo.add_and_configure_ipfabric_on_switch
parameters:
  fabric: new_fabric
  host: 10.254.10.73
status: succeeded (184s elapsed)
start_timestamp: 2016-09-14T00:24:38.075697Z
end_timestamp: 2016-09-14T00:27:42.673688Z
+-------------------------------+--------------------------+------------------------+------------------------+----------------------+
| id                            | status                   | task                   | action                 | start_timestamp      |
+-------------------------------+--------------------------+------------------------+------------------------+----------------------+
|   57d898c618971279ffcd65b1    | succeeded (6s elapsed)   | add_switch             | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | topology.switch_add    | 00:24:38 UTC         |
| + 57d898cc18971279ffcd65b5    | succeeded (173s elapsed) | configure_fabric       | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _fabric                | 00:24:44 UTC         |
|    57d898ce18971279ffcd65b7   | succeeded (8s elapsed)   | get_fabric_topology    | bwc-ipfabric.query_top | Wed, 14 Sep 2016     |
|                               |                          |                        | ology                  | 00:24:46 UTC         |
|  + 57d898d718971279ffcd65b9   | succeeded (77s elapsed)  | configure_switches     | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch                | 00:24:55 UTC         |
|   + 57d898da18971279ffcd65c9  | succeeded (35s elapsed)  | configure_interfaces   | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_ifaces         | 00:24:58 UTC         |
|      57d898dd18971279ffcd65d4 | succeeded (7s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:01 UTC         |
|      57d898e518971279ffcd65e1 | succeeded (5s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:09 UTC         |
|      57d898eb18971279ffcd65e7 | succeeded (8s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:15 UTC         |
|      57d898f418971279ffcd65f1 | succeeded (5s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:24 UTC         |
|   + 57d898ff18971279ffcd6607  | succeeded (33s elapsed)  | configure_bgp          | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_bgp            | 00:25:35 UTC         |
|      57d8990118971279ffcd6609 | succeeded (6s elapsed)   | configure_bgp          | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_bgp | 00:25:37 UTC         |
|      57d8990918971279ffcd6619 | succeeded (7s elapsed)   | configure_bgp_redistri | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | buted_routes           | _bgp_redistribute      | 00:25:45 UTC         |
|      57d8991118971279ffcd6625 | succeeded (8s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:53 UTC         |
|      57d8991118971279ffcd6627 | succeeded (8s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:53 UTC         |
|      57d8991218971279ffcd6628 | succeeded (8s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:53 UTC         |
|      57d8991218971279ffcd6629 | succeeded (9s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:53 UTC         |
|  + 57d898d718971279ffcd65c1   | succeeded (72s elapsed)  | configure_switches     | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch                | 00:24:55 UTC         |
|   + 57d898d918971279ffcd65c7  | succeeded (26s elapsed)  | configure_interfaces   | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_ifaces         | 00:24:57 UTC         |
|      57d898dd18971279ffcd65d3 | succeeded (6s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:01 UTC         |
|      57d898e318971279ffcd65d9 | succeeded (5s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:07 UTC         |
|      57d898e918971279ffcd65e3 | succeeded (5s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:13 UTC         |
|      57d898ef18971279ffcd65ed | succeeded (4s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:19 UTC         |
|   + 57d898f518971279ffcd65f3  | succeeded (37s elapsed)  | configure_bgp          | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_bgp            | 00:25:25 UTC         |
|      57d898f618971279ffcd65f5 | succeeded (6s elapsed)   | configure_bgp          | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_bgp | 00:25:26 UTC         |
|      57d898ff18971279ffcd6605 | succeeded (5s elapsed)   | configure_bgp_redistri | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | buted_routes           | _bgp_redistribute      | 00:25:34 UTC         |
|      57d8990718971279ffcd660f | succeeded (7s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:43 UTC         |
|      57d8990718971279ffcd6611 | succeeded (13s elapsed)  | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:43 UTC         |
|      57d8990818971279ffcd6617 | succeeded (8s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:43 UTC         |
|      57d8990818971279ffcd6616 | succeeded (10s elapsed)  | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:43 UTC         |
|  + 57d898d718971279ffcd65bc   | succeeded (77s elapsed)  | configure_switches     | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch                | 00:24:55 UTC         |
|   + 57d898d918971279ffcd65c5  | succeeded (28s elapsed)  | configure_interfaces   | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_ifaces         | 00:24:57 UTC         |
|      57d898dd18971279ffcd65d5 | succeeded (7s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:01 UTC         |
|      57d898e518971279ffcd65e0 | succeeded (6s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:09 UTC         |
|      57d898ed18971279ffcd65e9 | succeeded (6s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:17 UTC         |
|   + 57d898f918971279ffcd65fa  | succeeded (34s elapsed)  | configure_bgp          | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_bgp            | 00:25:29 UTC         |
|      57d898fb18971279ffcd65fe | succeeded (8s elapsed)   | configure_bgp          | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_bgp | 00:25:31 UTC         |
|      57d8990518971279ffcd660d | succeeded (5s elapsed)   | configure_bgp_redistri | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | buted_routes           | _bgp_redistribute      | 00:25:40 UTC         |
|      57d8990b18971279ffcd661d | succeeded (10s elapsed)  | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:47 UTC         |
|      57d8990b18971279ffcd661f | succeeded (10s elapsed)  | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:47 UTC         |
|     57d8991e18971279ffcd662d  | succeeded (3s elapsed)   | configure_anycast_gate | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | way                    | _anycast               | 00:26:06 UTC         |
|  + 57d898d718971279ffcd65be   | succeeded (58s elapsed)  | configure_switches     | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch                | 00:24:55 UTC         |
|   + 57d898d918971279ffcd65c3  | succeeded (17s elapsed)  | configure_interfaces   | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_ifaces         | 00:24:57 UTC         |
|      57d898db18971279ffcd65cd | succeeded (4s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:24:59 UTC         |
|      57d898e018971279ffcd65d7 | succeeded (3s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:04 UTC         |
|      57d898e518971279ffcd65dc | succeeded (4s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:09 UTC         |
|   + 57d898ee18971279ffcd65eb  | succeeded (22s elapsed)  | configure_bgp          | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_bgp            | 00:25:18 UTC         |
|      57d898f018971279ffcd65ef | succeeded (6s elapsed)   | configure_bgp          | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_bgp | 00:25:20 UTC         |
|      57d898f918971279ffcd65f8 | succeeded (4s elapsed)   | configure_bgp_redistri | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | buted_routes           | _bgp_redistribute      | 00:25:28 UTC         |
|      57d898fd18971279ffcd6601 | succeeded (5s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:33 UTC         |
|      57d898fd18971279ffcd6603 | succeeded (5s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:33 UTC         |
|     57d8990718971279ffcd6613  | succeeded (6s elapsed)   | configure_anycast_gate | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | way                    | _anycast               | 00:25:43 UTC         |
|  + 57d898d718971279ffcd65c0   | succeeded (72s elapsed)  | configure_switches     | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch                | 00:24:55 UTC         |
|   + 57d898db18971279ffcd65cb  | succeeded (26s elapsed)  | configure_interfaces   | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_ifaces         | 00:24:59 UTC         |
|      57d898dd18971279ffcd65cf | succeeded (6s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:01 UTC         |
|      57d898e518971279ffcd65dd | succeeded (6s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:08 UTC         |
|      57d898eb18971279ffcd65e6 | succeeded (7s elapsed)   | configure_interface    | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_ip  | 00:25:15 UTC         |
|   + 57d898f918971279ffcd65fb  | succeeded (29s elapsed)  | configure_bgp          | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _switch_bgp            | 00:25:29 UTC         |
|      57d898fb18971279ffcd65ff | succeeded (7s elapsed)   | configure_bgp          | bwc-                   | Wed, 14 Sep 2016     |
|                               |                          |                        | ipfabric.configure_bgp | 00:25:31 UTC         |
|      57d8990418971279ffcd660b | succeeded (4s elapsed)   | configure_bgp_redistri | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | buted_routes           | _bgp_redistribute      | 00:25:40 UTC         |
|      57d8990c18971279ffcd6621 | succeeded (9s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:47 UTC         |
|      57d8990c18971279ffcd6620 | succeeded (8s elapsed)   | configure_bgp_peers    | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          |                        | _bgp_neighbor          | 00:25:47 UTC         |
|     57d8991718971279ffcd662b  | succeeded (3s elapsed)   | configure_anycast_gate | bwc-ipfabric.configure | Wed, 14 Sep 2016     |
|                               |                          | way                    | _anycast               | 00:25:59 UTC         |
|    57d8992918971279ffcd662f   | succeeded (79s elapsed)  | show_bgp_config        | bwc-topology.show_conf | Wed, 14 Sep 2016     |
|                               |                          |                        | ig_bgp                 | 00:26:16 UTC         |
|   57d8997c18971279ffcd6631    | succeeded (2s elapsed)   | notify_success         | slack.chat.postMessage | Wed, 14 Sep 2016     |
|                               |                          |                        |                        | 00:27:40 UTC         |
+-------------------------------+--------------------------+------------------------+------------------------+----------------------+
vagrant@bwctest:~/aig_demo$
```
