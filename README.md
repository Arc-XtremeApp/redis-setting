# redis-setting

kubectl apply -f redis-conf.yml

kubectl apply -f redis-pv.yml

kubectl apply -f redis-sts.yml

kubectl apply -f redis-svc.yml


部屬完畢之後

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


# proxy 製作

# 下指令

yum install centos-release-scl

yum-config-manager --enable rhel-server-rhscl-7-rpms

yum install rh-python35

scl enable rh-python35 bash

yum install devtoolset-9

scl enable devtoolset-9 bash

wget https://github.com/RedisLabs/redis-cluster-proxy/archive/unstable.zip

unzip unstable.zip

make

make PREFIX=/usr/local/redis_cluster_proxy install

cp /usr/local/redis_cluster_proxy/bin/redis-cluster-proxy .

# 建立dockerfile

FROM centos:7

WORKDIR /data

ADD redis-cluster-proxy /usr/local/bin/

EXPOSE 7777

# build一下

docker build . -t redis-cluster-proxy:v1.0.0

# 打tag
docker tag redis-cluster-proxy:v1.0.0 arc119226/redis-cluster-proxy:v1.0.0

# 推到 docker hub
docker login
docker push arc119226/redis-cluster-proxy:v1.0.0

# 修改 cluster IP 為 redis svc 的 ip

kubectl apply -f proxy-conf.yml

# deploy

kubectl apply -f proxy-deploy.yml

# service

kubectl apply -f proxy-svc.yml


# test

延遲

redis-cli --intrinsic-latency 100

壓力測試

redis-benchmark -p 30001 -q -n 100000 -r 100000
