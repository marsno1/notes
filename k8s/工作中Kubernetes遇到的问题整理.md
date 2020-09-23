工作中Kubernetes遇到的问题整理：

1. Kubernetes 生产最佳实践：

   https://github.com/gravitational/workshop/blob/master/k8sprod.md

2. Event exporter: 

   https://github.com/GoogleCloudPlatform/k8s-stackdriver/tree/master/event-exporter

   https://git.jd.com/bag/event-exporter

3. scheduler性能优化：

   https://github.com/kubernetes/kubernetes/pull/63975

   https://github.com/kubernetes/kubernetes/pull/65714  ecahe

   Kube-mark相关:

   https://github.com/kubernetes/community/blob/master/contributors/devel/kubemark-guide.md

   https://github.com/kubernetes/community/blob/master/sig-scalability/tools/performance-comparison-tool.md

   https://github.com/kubernetes/community/blob/master/sig-scalability/goals.md

4. kubelet驱逐了pod之后，再恢复正常时，static pod没有被重启

   这个是设计这样做的： https://github.com/kubernetes/kubernetes/issues/40573

   Now onto static pods. Once a static pod gets evicted, it will not be re-started by a kubelet unless it restarts. When it restarts, the kubelet simply admits all static pods even ones that have been previously evicted (cc @yujuhong). As a result of this, a static pod that was once evicted and is not expected to be running will be admitted and a regular pod that was already running on the node might be booted from the node. This behavior is highly undesirable. Kubelet can look at the "mirror pod" for a static pod and decide not to recreate previously evicted static pods. But this is only possible if the kubelet has access to the apiserver before it processes static pods. Instead of attempting to solve this problem, we will instead attempt to avoid using static pods as much as possible in production. In addition, we need to update kubelet to only operate in one of two modes "local" or "remote". Once the kubelet switches to "remote" apiserver mode, it will stop accepting any "local" static (or http) pods. If the kubelet is explicitly switched back to "local" mode, it will evict all "remote"/unknown pods and only manage "local" pods. Upon switching from "local" to "remote", the kubelet will create mirror pod objects for "local" pods and then transfer control of those pods to the apiserver.

   所以我们要保证static pod不会被evict，甚至不要用static pod.

5.https://git.jd.com/bag/kubernetes/issues/115 kafka容器中看不到磁盘大小

   关注点：subpath和Mount Propagation的概念  https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath   https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation

6.dns解析5秒延迟 

https://blog.quentin-machu.fr/2018/06/24/5-15s-dns-lookups-on-kubernetes/ 详细描述

https://github.com/SumoLogic/sumologic-kubernetes-collection/pull/304 临时解决办法

7. https://git.jd.com/bag/kubernetes/issues/120 k8s并发问题

   很可能是多个deploymeny每个deployment一个pod，导致的controller性能问题。

8. 关于K8S接口调用测试的结果 bufflo团队 https://git.jd.com/bag/kubernetes/issues/134
  
  知识点：以1号进程启动的进程，若该进程没有自定义信号处理程序的话，内核想不做任何处理，
  
  所以我们需要给docker将上dummy-init程序来启动容器内的程序。

9. kubelet错误日志: Partial failure issuing cadvisor.ContainerInfoV2 https://git.jd.com/bag/kubernetes/issues/137
   https://bugzilla.redhat.com/show_bug.cgi?id=1370299
   https://github.com/docker/runc/blob/17.03.x/libcontainer/cgroups/systemd/apply_systemd.go#L163
   useSystemd的测试导致该问题
   https://gobomb.github.io/post/container-runtime-note/
   https://feisky.gitbooks.io/kubernetes/plugins/CRI.html

10. 重启kubelet报临时存储错误： 该 pr https://github.com/kubernetes/kubernetes/pull/59769 修复

11. 实现local PV的自动化管理：https://git.jd.com/bag/kubernetes/issues/151
    https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/
    local pv 与 hostPath 区别 local pv可以和某个node绑定，从而使用某个pv的pod同样会被调度到该node上。

12. kc1集群有的pod删除不了 
    kernel 3.10 的bug, 节点上，systemd控制的service文件中如果有服务配置了PrivateTmp=true，容器在销毁时需要卸载挂载的磁盘信息，这样就会产生问题，因为容器的挂在信息会泄漏到配置了PrivateTmp=true的服务挂在信息中（bug）
    https://blog.frognew.com/2018/02/k8s-unable-delete-pod-docker-device-busy.html
    https://ekuric.wordpress.com/2015/10/09/docker-complains-about-cannot-remove-device-or-resource-busy/

13. 建立对kubelet的外部健康检查机制。https://git.jd.com/bag/kubernetes/issues/178
    最后的实现是通过controller-manager定时从prometheus获取节点的健康指标，双向判断kubelet是否not ready，从而决定时候需要evict pods.
    
14. rbd pvc 挂载目录错误 https://git.jd.com/bag/kubernetes/issues/185
    由于/data/kubelet,/data/kubelet/pods 目录中包含了额外的挂载点所导致，例如data9挂在到了data，这样就会导致卸载data中包含了额外的挂载点。解决办法，data与data9全部换成private，不共享挂在信息。 (kubelet代码里限制不允许kubelet的工作目录被其他目录挂载)

15. Docker环境的Page Cache 
    Container使用的page cache是计算到container的内存使用中的。如果这个Container需要使用很大的page cache，请在内存资源申请中加上。
    https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-memory （官方cgroup文档）

16. docker lxcfs 的支持
    容器中查看top free不会看到物理机的数据
    http://www.qingpingshan.com/pc/fwq/32337.html
    http://blog.csdn.net/shida_csdn/article/details/79196258
    https://www.kubernetes.org.cn/1423.html

17. Container下jvm参数调整 https://developers.redhat.com/blog/2017/04/04/openjdk-and-containers/

18. 文生有先见之明让推广operator模式:

    https://coreos.com/blog/introducing-operators.html

    kafka-operator: https://github.com/krallistic/kafka-operator 

    etcd-operator: https://github.com/coreos/etcd-operator 

    prometheus operator: https://github.com/coreos/prometheus-operator

19. 当DaemonSet sync有错误时DaemonSetsController耗费大量CPU

    DaemonSetsController.simulate会模拟是否成功调度到node上，它会传递newPod,以及nodeInfo到predcates来做资源验证，它在创建nodeInfo的时候会计算node上已经存在多少个pod，但是这个运算是将集群中所有的pod查出来，逐个筛选，我看了下监控目前有2w+的pod（这也涉及到growslice等内存占用问题）。而且informer使用的是shareInformer,这是共享Informer，所有controller都会公用的cache，所以频繁使用也会存在锁的问题。

    官方社区已经解决这个问题，给每个pod加了索引，具体请看下https://github.com/kubernetes/kubernetes/pull/65009

20. 开发了SelectorPinpack策略 https://git.jd.com/bag/kubernetes/merge_requests/17

    SelectorSpreading priority policy （plugin/pkg/scheduler/algorithm/priorities/selector_spreading.go）会试图将同一类的POD分散到不同的node和availability zone上。

    我们现在的zone定义为一个物理POD。这个策略本来是为了将worker分散到不同的zone里面，以提高job的可靠性。然而对于类似storm的job来说，worker是一个整体，缺一不可。将同一job的worker分散到不同的zone里面，无法提高可靠性，反而会增加跨物理POD的网络流量。

    我们应该修改调度策略，使得同一个job的不同worker尽量调度到同一个zone里面

21. gc controller回收已经退出的POD应该考虑POD中容器的退出状态  

    https://git.jd.com/bag/kubernetes/merge_requests/18 优先回收退出状态为成功的pod

22. https://www.oschina.net/translate/docker-and-the-pid-1-zombie-reaping-problem?lang=chs&p=2

    docker的容器需要init进程进行僵尸回收

23. https://git.jd.com/bag/kubernetes/issues/264 借鉴VIP在k8s方面的一些实践 (一定要看)

24. https://git.jd.com/bag/design-proposals/merge_requests/1 
    混合部署，我理解需要考虑不同业务混合部署后需要考虑到资源能够尽量的做到强隔离，从而避免不同业务的互相影响：
    kernel memory的隔离（slab, buffer, etc）, 因为统计pod的使用时候cadvisor是需要计算这些cache的。
    
    Node上PoD eviction policy需要进一步研究清楚
    容器的标准配置问题：kubernetes#275 (closed)
    kernel memory的隔离 (slab, buffer，etc）
    Memory request/limit如何更好的起作用
    CPU的隔离：guarantee POD能尽快拿到CPU
    disk IO和network IO的隔离
    网络流量的QoS
    
    https://github.com/intel/platform-resource-manager
    https://github.com/intel/owca
    Intel 开源的两个关于混部的项目，主要目标是解决单机混部不同类型 workload 时资源抢占的问题。
    应该是根据优先级 进行 Cgroup 的动态调节，据介绍调节内容包含 CPU share、Quota、Cache，及 Memory bandwidth等。未调节Period 参数。
    目前也没有磁盘相关的控制，需要依赖Cgroup v2。
    该项目对内核版本和CPU型号有要求。代码为Python代码，有时间再研究一下。

    https://git.jd.com/bag/design-proposals/blob/co-locate-scheduling/kubernetes/scheduling/co-locate.md （把这个详细看一下）


25. 多调度器：https://git.jd.com/bag/design-proposals/blob/multi-schedulers/kubernetes/multi-schedulers.md

26. in placec update https://github.com/kubernetes/community/pull/1719 https://github.com/kubernetes/enhancements/pull/686

27. controller-manager性能优化 https://git.jd.com/bag/kubernetes/merge_requests/117/diffs
    Improve performance for scheduler getSelector function https://git.jd.com/bag/kubernetes/merge_requests/113/diffs

28. https://github.com/kubernetes/node-problem-detector 调研