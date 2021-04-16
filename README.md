# Kubernetes Cheat Sheet
Kubernetes Cheat Sheet with the most needed stuff..














# kubectl

<br><br><br><br>





## Logs

<br><br>

#### Get logs from pod
```bash
kubectl -n green logs sample-1234abcd
```

<br><br>

#### Get logs from pod with multiple container
```bash
kubectl -n green logs sample-1234abcd -c sample
```



<br><br>
__________________________________________________________
__________________________________________________________
<br><br>




# k9s

<br><br>



## Deploy

<br><br>

#### See current deployments
```bash
k9s
:deploy
```

#### Restart deployment
```bash
k9s
:deploy

# Now choose your deployment and press s. Now you can set the replica to 0 and it will shutdown. Then after this set it again to the amount of replicas which it has before. Then it will restart.
```





<br><br><br><br>

## Logs

<br><br>

#### Logs not showing in Pod
- You can press 0 or:
```bash
~/.k9s/config.yml
k9s.logger.sinceSeconds=-1
```
