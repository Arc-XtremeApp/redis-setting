# redis-setting
redis in k8s
在k8s上設定

# 找出pod IP 會得到 IP:6379
kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 '

# 執行redis cluster 部署
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(echo "172.17.0.4:6379 172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379")


# 驗證群集狀態
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli role; echo; done

#重置

# 刪除pvc
for x in $(seq 0 5); do echo "data-redis-cluster-$x"; kubectl delete pvc data-redis-cluster-$x; echo; done

# 更新權限
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli acl load; echo; done


