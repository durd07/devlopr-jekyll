---
pytitle: logbot
---

# LogBot

## dashboard

[http://135.252.135.253:31930/](http://135.252.135.253:31930/)

[LogBot - QD TA - Dashboard (nokia.com)](https://confluence.ext.net.nokia.com/display/QDTA/LogBot)

## anomaly score system

1. install dependency

	```sh
	export PYTHONPATH=/root/src/machine-learning/ntas-ml-super-project/ntas_anomaly_detection/src

	pip install numpy pyyaml matplotlib pandas click openpyxl memory-profiler
	```

2. configuration and file location
	- [/etc/logbot-config/logbot_config.yaml.base64](/uploads/logbot/etc/logbot-config/logbot_config.yaml.base64)
	- [ /etc/anomaly_score_res/ntas_mcs_marker.yaml](/uploads/logbot/etc/anomaly_score_res/ntas_mcs_marker.yaml)
	- [/root_fs/home/nfs_volume/base_path/spark_test/data/10w.log.original_line_number_to_cluster_id_data.npy](/uploads/logbot/data/spark_test/data/10w.log.original_line_number_to_cluster_id_data.npy)
	- [/root_fs/home/nfs_volume/base_path/spark_test/data/10w.log.cluster_data.npy](/uploads/logbot/data/spark_test/data/10w.log.cluster_data.npy)
	- [/root_fs/home/nfs_volume/base_path/spark_test/data/task123_10w.log.frequency.pkl](/uploads/logbot/data/spark_test/data/task123_10w.log.frequency.pkl)

3. output:
	- [/root_fs/home/nfs_volume/base_path/task123_10w.log.xlsx]()

4. run:

	```sh
	python3 ./src/log_anomaly_score_judge.py --log_file_name 10w.log --task_name task123
	```

> kubectl exec -it logbot-controller-5bb559557-g5t6s bash -n logbot  
> /root_fs/home/nfs_volume/mocui/ntas-ml-super-project/log_analytics/log_anomaly_score_system/src# ./log_anomaly_score_judge.py --log_file_name 10w.log --task_name task123

## TODO

把PM匹配的功能加入logbot:

1. 添加参数"--pm-data-dir"
2. 读取所有的pm并插入es; 你需要建立一个新的index，design一个新的map，然后准备一个新的Kibana search
3. 找到所有匹配的log cluster
4. 将结果插入cluster_level_anomaly excel sheet; 这里要有kibana_dashboard，等你把pm数据插入es，我们再design dashboard



```
CONFIG_FILE_PATH=/root_fs/home/nfs_volume/mocui/ntas-ml-super-project/log_analytics/logbot/src/test/config/logbot_config.yaml 
```

# Technical Stack

## Argo

https://argoproj.github.io/

Argo UI page

https://135.252.135.230:3274/

## SparkOperator

https://github.com/GoogleCloudPlatform/spark-on-k8s-operator

```sh
# submit argo workflow to start logbot
argo submit logbot_workflow.yaml --parameter-file tonyawparams.yaml -v -k --name <task_name>
argo -v -k list
argo -v -k get @latest
STEP                      TEMPLATE          PODNAME                    DURATION  MESSAGE
 ● logbot-felixdu         main
 ├───○ cn-log-conversion  cn-log-adapter                                         when 'vnf == cn' evaluated false
 └───● logbot-main-flow   logbot-main-flow  logbot-felixdu-1427400815  3m
 kubectl logs logbot-felixdu1-1427400815 -n logbot -c log-clustering elasticsearch-insertion frequency-calculation anomaly-score-calculation
 
# show sparkapplications
kubectl get sparkapplication -n logbot
```



## Python To .so

```python
from distutils.core import setup
from Cython.Build import cythonize
setup (
     ext_modules = cythonize(' .pyx')
)

python a.py build_ext --inplace
```



**apt-get install gdb python3.7-dbg  gdb python3 62然后py-btpy-list用gdb debug python**



fp-growth

https://towardsdatascience.com/understand-and-build-fp-growth-algorithm-in-python-d8b989bab342
