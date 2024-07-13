# Istio Study Document

## 个人学习关键点梳理

### Istio Sidecar 自动注入

1. 首先需要给命名空间(如默认命名空间)添加标签，指示 Istio 在部署应用的时候，自动注入 Envoy 边车代理
~~~
kubectl label namespace default istio-injection=enabled
~~~

2. 要验证标签是否已成功添加到default命名空间，你可以运行以下命令来列出命名空间的标签：
~~~
kubectl get namespace default --show-labels
~~~

3. Istio的自动注入机制
* Sidecar注入：Istio通过向Pod中注入一个Sidecar代理（默认是Envoy）来拦截Pod的入站和出站流量，从而实现流量管理、安全、可观察性等功能。
* 自动注入条件：Istio通过检查命名空间的标签来决定是否自动向该命名空间中的Pod注入Sidecar。当命名空间被标记为istio-injection=enabled时，Istio会自动向该命名空间中的新Pod注入Sidecar。
* 实现方式：Istio使用Mutating Admission Webhook（变更准入Webhook）来监听Kubernetes API Server中的Pod创建事件。当检测到Pod创建事件时，如果Pod所在的命名空间被标记为istio-injection=enabled，Istio会修改Pod的定义，向其中注入Sidecar代理。

 ```
Mutating Admission Webhook（变更准入Webhook） 是Kubernetes中一种强大的准入控制机制，它允许在Kubernetes对象（如Pods、Services等）被创建或更新之前，对它们进行修改。
这种机制通过HTTP回调的方式，允许开发者编写自定义的代码来拦截并处理到达Kubernetes API Server的请求。
一、工作原理
当Kubernetes API Server接收到一个创建或更新资源的请求时，如果该资源配置了Mutating Admission Webhook，则API Server会将请求发送给Webhook服务。
Webhook服务根据自定义的逻辑对请求中的对象进行修改，并将修改后的对象返回给API Server。API Server在接收到修改后的对象后，会将其存储到etcd中。
二、主要特点
自定义性强：开发者可以根据业务需求，编写自定义的Webhook服务，对Kubernetes对象进行个性化的修改。
灵活性高：由于Webhook服务是独立于Kubernetes API Server运行的，因此它可以根据需要部署在任何地方，包括Kubernetes集群内部或外部。
可扩展性好：通过添加新的Webhook配置，可以轻松扩展Kubernetes的准入控制功能，而无需修改Kubernetes本身的代码。
 ```

**总结kubectl label namespace default istio-injection=enabled 命令的工作原理**
* 执行命令：通过运行kubectl label namespace default istio-injection=enabled命令，用户为default命名空间添加了一个istio-injection=enabled的标签。
* Istio响应：Istio的Mutating Admission Webhook会监听到命名空间的这一变化，并记住哪些命名空间被标记为允许自动注入。
* 自动注入：当在default命名空间中创建新的Pod时，Istio的Webhook会拦截这一事件，并检查Pod所在的命名空间是否允许自动注入。如果允许（即命名空间被标记为istio-injection=enabled），Istio会修改Pod的定义，向其中注入Envoy Sidecar代理。

