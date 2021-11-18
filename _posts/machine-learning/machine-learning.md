---
title: Machine Learning
---

# Machine Learning

## ML
[SVMSVC](https://blog.csdn.net/qq_41577045/article/details/79859902)

https://colab.research.google.com/


## TeX
- [online LaTex (overleaf)](https://www.overleaf.com/)
- [TeX Live](https://www.tug.org/texlive/)
- [arXiv.org](https://arxiv.org/)

## Links
[10 minutes to pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/10min.html)


## Info

https://nokia.sharepoint.com/sites/qingdaotechnologycenter/SitePages/Nokia-Qingdao-Machine-Learning-2019-Code-Jam.aspx

[Code Jam kickoff](https://nokia.sharepoint.com/:p:/r/sites/qingdaotechnologycenter/_layouts/15/Doc.aspx?sourcedoc=%7B857E3C30-35DE-4F8B-A7E8-A812DA1573CC%7D&file=Codejam-kickoff.pptx&action=edit&mobileredirect=true)
BaseLine https://github.com/mdeff/fma

Tips:
Only recognize top genres
Use data in fma_metadata.zip:?
  - features.csv - features
  - tracks.csv - make use of below columns:
    - track.genre_top: Top genres of each track, i.e., our class labels in code jam?
    - set.subset: Marks of subset (small/medium/large/full). Use medium subset in code jam, i.e., subset <= medium?
    - set.split: Marks of training/validation/testing data: 19922 training examples, 2505 validation examples, 2573 testing examples.?

Submission - two Jupyter notebooks:
- One notebook loads training/validation data, perform training, save the model, and print Precision, Recall and F1-score of the training/validation data.
- Another notebook loads the model and predict the testing data, print the Precision, Recall and F1-score.

For challenge 2, please print Precision, Recall and F1-score of every class, as well as macro-averaged F1-score.

Document your procedure/code clearly in the notebook.
List all the references in the submission, e.g., paper, web link
Upload the submission as one zip/tar file to the directory Code Jam/2019 on the sharepoint. Name the file as <team-name>_challenge[1/2].zip (e.g., MyTeam_challenge1.zip)?

Data can be downloaded from http://harold-xue.dynamic.nsn-net.net/
Hadoop information:
-	1 master node: hadoop@10.129.65.45 (password: hadoop), HADOOP_HOME: /usr/hadoop/Hadoop.
-	4 slave node, slave1, slave2, slave3, slave4 which are configured in /etc/hosts on master node. (You can also see master2, but it’s not in use yet)
-	400 GB Cinder volume is mounted on master node as nameNode data storage. And each slave node has 2.8 TB volume as dataNode storage.
-	HDFS webui: http://10.129.65.45:9870/
-	Hadoop webui: http://10.129.65.45:8088/


The refactored training code of our ML project is committed to the branch TMROBOT-590. Please note that it is partially finished. For example, it only support one preprocess pipeline, one classifier.
TMROBOT-590


Servers
10.164.201.109, root/xcy240325
https://delfin.bell-labs.com/
RRT Aiding System http://10.164.201.109/
Source Code: https://gitlabe2.ext.net.nokia.com/haxue/ntas-ci-tools


lemmatizer.lemmatize

tfidf

时间 筛选当前case的container log

compare passwd and failed log

正则化，过拟合

多分类

网格搜索
随机搜索


全连接神经网络
CNN/RNN


采样分析效果

