REST是一套接口开发原则， 除了六个约束外， 其他不做规定；
六个约束： 
1. 统一的接口

 定义了客户端和服务端的接口。
 遵循四个原则： 
 基于资源的；URIs作为资源标识符，来区分不同的资源； 将服务端资源转化成JSON等格式给到客户端。
 客户端可以修改服务端资源。
 自我描述： 每条消息都包含足够的信息来描述如何处理该消息。
客户端通过正文内容、查询字符串参数、请求头和请求的URI（资源名称）传递状态。服务通过正文内容、响应代码和响应头将状态传递给客户端。


2. 无状态的
处理请求所需的状态包含在请求本身中，无论是作为URI、查询字符串参数、正文还是请求头的一部分。
然后，在服务器进行处理之后，适当的状态通过头、状态和响应体传回客户端。
这样就摆脱了会话状态， 服务的可伸缩性加强了， 支持集群部署。

3. 可缓存的
服务端必须明确指定返回内容可否缓存。

4. cs架构
client和server是分离的，并通过接口沟通。

5. 支持分层系统
客户端和服务端之间还可以添加其他层， 比如反向代理和负载均衡器，缓存层等。

6. 按需编码（可选的）
服务器可以通过将逻辑传输到可以执行的客户端，临时扩展或自定义客户端的功能。

 tips
用好http verbs
定义合理的接口名称和合理的路径定义
用JSON等格式传递数据
创建细粒度合适的接口
考虑接口的连通性

 名词定义
幂等这个术语被更全面地用来描述一个操作，如果执行一次或多次，将产生相同的结果。
PUT和DELETE方法被定义为幂等的；
GET、HEAD、OPTIONS和TRACE方法也被定义为幂等的，因为它们被定义为安全的。


 Http verbs
主要或最常用的HTTP动词（或方法，因为它们被正确地调用）是POST、GET、PUT和DELETE。它们分别对应于创建、读取、更新和删除（或CRUD）操作。

PUT最常用于更新功能，请求请求体包含原始资源的最新更新表示。
但是，在客户端而不是服务器选择资源ID的情况下，PUT也可以用于创建资源。
换句话说，如果PUT属于包含不存在的资源ID.的值的URI，则请求体包含资源表示。
许多人认为这是复杂和混乱的。因此，这种创造的方法应该尽量少用，如果有的话。
更新成功后，从PUT返回200（如果不返回正文中的任何内容，则返回204）。如果使用PUT进行创建，则在成功创建时返回HTTP状态201。


post
成功创建时，返回HTTP状态201，

delete 
成功删除时，返回HTTP状态200（OK）和响应体，可能是已删除项的表示形式。或者返回没有响应体的HTTP状态204（没有内容）。
换句话说，建议使用204状态（没有正文）

从HTTP规范来看，删除操作是幂等的。如果删除资源，它将被删除。
然而，关于删除等幂有一个警告。第二次对资源调用DELETE通常会返回404（未找到），因为它已被删除，因此不再可找到。


HTTP Verb
/customers
/customers/{id}


GET
200 (OK), list of customers. Use pagination, sorting and filtering to navigate big lists.
200 (OK), single customer. 404 (Not Found), if ID not found or invalid.


PUT
404 (Not Found), unless you want to update/replace every resource in the entire collection.
200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid.

POST 
POST 201 (Created), 'Location' header with link to 404 (Not Found). /customers/{id} containing new ID.
404 (Not Found).

DELETE
404 (Not Found), unless you want to delete the whole collection—not often desirable.
200 (OK). 404 (Not Found), if ID not found or invalid.

资源命名

过滤和分页

接口版本升级


