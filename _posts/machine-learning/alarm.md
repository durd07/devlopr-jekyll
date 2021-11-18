---
title: NTAS Machine Learning
---

# Alarm

## Teams Channel
- [TAS ML New Force]()
- [NTAS Alarm Correlation](https://teams.microsoft.com/l/file/7cc0a575-f01a-4d7f-9111-ca42f0460bf0?tenantId=5d471751-9675-428d-917b-70f44f9630b0&fileType=xlsx&objectUrl=https%3A%2F%2Fnokia-my.sharepoint.com%2Fpersonal%2Ffu_tang_wang_nokia-sbell_com%2FDocuments%2FMicrosoft%20Teams%20Chat%20Files%2FReference_Alarms_Nokia_TAS_20210120.xlsx&baseUrl=https%3A%2F%2Fnokia-my.sharepoint.com%2Fpersonal%2Ffu_tang_wang_nokia-sbell_com&serviceName=p2p&threadId=19:4e8d4771720a4b0ab2948859a03bcacf@thread.v2)
- [NTAS Alarm Correlation Knowledge Graph](https://confluence.ext.net.nokia.com/display/QDTA/NTAS+Alarm+Correlation+Knowledge+Graph)

## Code

- [https://gitlabe1.ext.net.nokia.com/mocui/ntas-ml-super-project](https://gitlabe1.ext.net.nokia.com/mocui/ntas-ml-super-project)
- [https://gitlabe1.ext.net.nokia.com/tonyaw/log_analytics](https://gitlabe1.ext.net.nokia.com/tonyaw/log_analytics)
- [Zhang Tiana / logbot_build · GitLab (nokia.com)](https://gitlabe1.ext.net.nokia.com/tizhang/logbot_build)
- [Wang Tony H / ntas_anomaly_detection · GitLab (nokia.com)](https://gitlabe1.ext.net.nokia.com/tonyaw/ntas_anomaly_detection)
- [Zhang Tiana / logbot_deliver · GitLab (nokia.com)](https://gitlabe1.ext.net.nokia.com/tizhang/logbot_deliver)
> - code for ready from csv and import to neo4j
>   log_analytics/knowledge_graph/src/dashboard/knowledge_collector_app/neo4j_manager.py
> - spark drain entrance  
>   log_analytics/log_cluster/src/log_clusterer/spark/drain_driver.py


## Server

iLO: Administrator/foss.newsys  
host: ssc: emerUs3r!
         root: Newsys0!


| NAME           | STATUS    | ROLES     | AGE     | VERSION    | INTERNAL-IP        | EXTERNAL-IP    | OS-IMAGE                 | KERNEL-VERSION                 | CONTAINER-RUNTIME | FLAVOR |
| -------------- | --------- | --------- | ------- | ---------- | ------------------ | -------------- | ------------------------ | ------------------------------ | ----------------- | ------ |
| foss-ssc-10    | Ready     | <none>    | 86d     | v1.18.8    | 135.252.135.250    | <none>         | CentOS Linux 7 (Core)    | 5.1.11-1.el7.elrepo.x86_64     | docker://20.10.2  | 8/32   |
| foss-ssc-13    | Ready     | master    | 220d    | v1.18.8    | 135.252.135.253    | <none>         | CentOS Linux 7 (Core)    | 3.10.0-957.el7.x86_64          | docker://19.3.12  | 40/128 |
| foss-ssc-14    | Ready     | <none>    | 220d    | v1.18.8    | 135.252.135.240    | <none>         | CentOS Linux 7 (Core)    | 3.10.0-957.el7.x86_64          | docker://19.3.12  | 40/128 |
| foss-ssc-8     | Ready     | <none>    | 23d     | v1.18.8    | 135.252.135.248    | <none>         | CentOS Linux 7 (Core)    | 5.9.6-1.el7.elrepo.x86_64      | docker://19.3.8   | 32/64  |
| foss-ssc-9     | Ready     | <none>    | 86d     | v1.18.8    | 135.252.135.249    | <none>         | CentOS Linux 7 (Core)    | 3.10.0-1127.19.1.el7.x86_64    | docker://19.3.13  | 24/128 |
| qd-graphics    | Ready     | <none>    | 220d    | v1.18.8    | 135.252.135.230    | <none>         | CentOS Linux 7 (Core)    | 5.8.1-1.el7.elrepo.x86_64      | docker://19.3.8   | 8/64   |


| server         | iLO             | IP              | oam          | ctro         |
| -              | -               | -               | -            | -            |
| foss-scc-1-G7  | 135.252.135.231 | 135.252.135.241 | 10.9.249.193 | 10.9.249.225 |
| foss-scc-2-G7  | 135.252.135.232 | 135.252.135.242 | 10.9.249.194 | 10.9.249.226 |
| foss-scc-3-G7  | 135.252.135.233 | 135.252.135.243 | 10.9.249.195 | 10.9.249.227 |
| foss-scc-4-G7  | 135.252.135.234 | 135.252.135.244 | 10.9.249.196 | 10.9.249.228 |
| foss-scc-5-G8  | 135.252.135.235 | 135.252.135.245 | 10.9.249.197 | 10.9.249.229 |
| foss-scc-6-G8  | 135.252.135.236 | 135.252.135.246 | 10.9.249.198 | 10.9.249.230 |
| foss-scc-7-G8  | 135.252.135.237 | 135.252.135.247 | 10.9.249.199 | 10.9.249.231 |
| foss-scc-8-G8  | 135.252.135.238 | 135.252.135.248 | 10.9.249.200 | 10.9.249.232 |
| foss-scc-9-G9  | 135.252.135.239 | 135.252.135.249 | 10.9.249.201 | 10.9.249.233 |
| foss-scc-10-G8 | 135.252.135.225 | 135.252.135.250 | 10.9.249.202 | 10.9.249.234 |
| foss-scc-11-G8 | 135.252.135.226 | 135.252.135.251 | 10.9.249.203 | 10.9.249.235 |
| foss-scc-12-G9 | 135.252.135.227 | 135.252.135.252 | 10.9.249.204 | 10.9.249.236 |
| foss-scc-13-G9 | 135.252.135.228 | 135.252.135.253 | 10.9.249.205 | 10.9.249.237 |
| foss-scc-14-G9 | 135.252.135.229 | 135.252.135.240 | 10.9.249.206 | 10.9.249.238 |
| qd-graphics    |                 | 135.252.135.230 | 10.9.249.207 | 10.9.249.239 |

[Kibana][1]

[1]: http://135.252.135.240:31930/app/dashboards#/view/74833020-9e7c-11eb-9376-1ddcb2b85287?_g=(filters:!(),refreshInterval:(pause:!t,value:0),time:(from:'2021-06-03T02:50:00.000000Z',to:'2021-06-03T05:00:00.000000Z'))&_a=(description:'',filters:!(('$state':(store:appState),meta:(alias:!n,controlledBy:'1618554248218',disabled:!f,index:'7a59ceb0-8625-11eb-9376-1ddcb2b85287',key:cluster_id,negate:!f,params:(query:585),type:phrase),query:(match_phrase:(cluster_id:585))),('$state':(store:appState),meta:(alias:!n,controlledBy:'1618970650484',disabled:!f,index:'7a59ceb0-8625-11eb-9376-1ddcb2b85287',key:task_name.keyword,negate:!f,params:(query:task0616-4g),type:phrase),query:(match_phrase:(task_name.keyword:task0616-4g)))),fullScreenMode:!f,options:(hidePanelTitles:!f,useMargins:!t),query:(language:kuery,query:''),timeRestore:!f,title:one_cluster_dashboard,viewMode:view)

## TAS Alarm Related Info

### Alarm Source Info:

https://scm.cci.nokia.net/ntas/sack/-/tree/master/app/ald/xml_form  
https://scm.cci.nokia.net/ntas/sack/-/tree/master/vspi/ald/xml_form

ECMS mi/sw/xml/ntas_alarm


## NTAS Alarm Scenario Supervised Learning  



# Neo4j
## Neo4j install APOC XLSX support

https://neo4j.com/labs/apoc/4.1/import/xls/
> need to manually change `http` schema to `https` when download jar files

add following configuration into `neo4j.conf`
```
apoc.import.file.enabled=true
```

```
CALL apoc.load.xls("file:///alarm.xlsx", "Sheet1")
YIELD map AS item
CREATE (a:Alarm {id: item.`Alarm number`, name: item.`Alarm name`, phase: item.phase, clearing: item.Clearing, category: item.Category, module: item.module, profile: item.profile})
RETURN a 
```

```
MATCH (n) DELETE n
```





