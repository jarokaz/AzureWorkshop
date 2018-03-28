# HDInsight Walkthroughs

## Use of SSH tunneling

The following Ambari Web UIs require an SSH tunnel:
- JobHistory
- NameNode
- Thread Stacks
- Oozie web UI
- HBase Master and Logs UI

Hue also requires SSH tunneling

### Create an SSH tunnel
```
ssh -C2qTnNf -D 9876 sshuser@CLUSTERNAME-ssh.azurehdinsight.net
```

## Check the default storage used by the cluster
```
curl -u admin -G "https://jkhdidemo1.azurehdinsight.net/api/v1/clusters/jkhdidemo1/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'
```

## Get the name of Data Lake Store
```
curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["dfs.adls.home.hostname"] | select(. != null)'
```

