# redis-setting
redis in k8s
在k8s上設定

# 找出pod IP
kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '

# redis cluster deploy
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(echo "172.17.0.4:6379 172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379")


# validate cluster
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli role; echo; done

#reset..

# delete pvc
for x in $(seq 0 5); do echo "data-redis-cluster-$x"; kubectl delete pvc data-redis-cluster-$x; echo; done

# update acl
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli acl load; echo; done


