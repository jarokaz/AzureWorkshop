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
ssh -C2qTnNf -D 9876 USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
```


