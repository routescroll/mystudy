# Explanation

## Common
- `Exponential backoff` (幂减)
  `Exponential backoff` can improve an application's reliability by using progressively longer waits between retries. (以幂减速度延长Retry时间)
  When using the `AWS SDK`, this `logic is built‑in`. (AWS SDK自带幂减处理逻辑)
  However, in this case the application is incompatible with the AWS SDK so it is necessary to manually implement exponential backoff. (不能用SDK的幂减逻辑只能在自己代码里实现该逻辑)

## Aurora
- Aurora AutoScaling策略：基于Average connections(平均连接数)
  > Average connections of Aurora Replicas which will create a policy based on the average number of connections to Aurora Replicas.

## CodeDeploy
- Deployment type<br>

  |TST Linear|TST Canary|TST All at once|Service|DPT Inplace|DPT Blue/green|
  |-|-|-|-|-|-|
  |-|-|-|EC2/On-Premises|◎|◎|
  |◎|◎|◎|Lambda|-|◎|
  |◎|◎|◎|ECS|-|◎|

- `AppSpec File`
  - `files section (EC2/On-Premises only)`
    > files:<br>
    &ensp;&ensp;- source: source-file-location-1<br>
    &ensp;&ensp;&ensp;&ensp;destination:destination-file-location-1<br>
    &ensp;&ensp;- source: source-file-location-2<br>
    &ensp;&ensp;&ensp;&ensp;destination:destination-file-location-2
file_exists_behavior: DISALLOW|OVERWRITE|RETAIN
  - `permission section (EC2/On-Premises only)`
    > permissions:<br>
    &ensp;&ensp;- object: object-specification<br>
    &ensp;&ensp;&ensp;&ensp;pattern: pattern-specification<br>
    &ensp;&ensp;&ensp;&ensp;except: exception-specification<br>
    &ensp;&ensp;&ensp;&ensp;owner: owner-account-name<br>
    &ensp;&ensp;&ensp;&ensp;group: group-name<br>
    &ensp;&ensp;&ensp;&ensp;mode: mode-specification<br>
    &ensp;&ensp;acls:<br>
    &ensp;&ensp;&ensp;&ensp;- acls-specification<br>
    &ensp;&ensp;context:<br>
    &ensp;&ensp;&ensp;&ensp;user: user-specification<br>
    &ensp;&ensp;&ensp;&ensp;type: type-specification<br>
    &ensp;&ensp;&ensp;&ensp;range: range-specification<br>
    &ensp;&ensp;type:<br>
    &ensp;&ensp;&ensp;&ensp;- object-type<br>
  - `resources section (ECS & Lambda only)`
    > resources:<br>
    &ensp;&ensp;- name-of-function-to-deploy:<br>
    &ensp;&ensp;&ensp;&ensp;type: "AWS::Lambda::Function"<br>
    &ensp;&ensp;&ensp;&ensp;properties:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;name: name-of-lambda-function-to-deploy<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;alias: alias-of-lambda-function-to-deploy<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;currentversion: version-of-the-lambda-function-traffic-currently-points-to<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;targetversion: version-of-the-lambda-function-to-shift-traffic-to<br>

  - `hooks Section`
    - The '`hooks`' section for an `EC2/On-Premises` deployment contains mappings that link deployment lifecycle event hooks to one or more scripts.
    - The '`hooks`' section for a `Lambda` or an `Amazon ECS` deployment specifies Lambda validation functions to run during a deployment lifecycle event. 
    - `Hook` section is `required only` if you are running scripts or Lambda validation functions as part of the deployment.
    - ECS Hook Order(Life Cycle Event)<br>
      ![ECS](https://routescroll.github.io/lifecycle-event-order-ecs.png)
    - Lambda Hook Order(Life Cycle Event)<br>
      ![Lambda](https://routescroll.github.io/lifecycle-event-order-lambda.png)
    - EC2/On-Premises Hook Order In-Place(Life Cycle Event)<br>
      ![ec2-in-place](https://routescroll.github.io/lifecycle-event-order-in-place.png)
    - EC2/On-Premises Hook Order Blue/Green(Life Cycle Event)<br>
      ![ec2-blue-green](https://routescroll.github.io/lifecycle-event-order-blue-green.png)

- 通过创建多个Deployment Group来实现多Stage分别Deploy

## CodeBuild
- buildspec **<font color=red >post_build</font>**
  - 可选项
  - 在Build Phase后执行,用于发布Build后的jar/war或Docker Image等
  - **<font color=red >finall block</font>** 中的命令无论 **<font color=red >command block</font>** 的执行结果是否成功都会执行
- buildspec sample
  > version: 0.2<br>
    run-as: Linux-user-name<br>
    env:<br>
    &ensp;&ensp;&ensp;&ensp;shell: shell-tag<br>
    &ensp;&ensp;&ensp;&ensp;variables:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key: "value"<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key: "value"<br>
    &ensp;&ensp;&ensp;&ensp;parameter-store:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key: "value"<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key: "value"<br>
    &ensp;&ensp;&ensp;&ensp;exported-variables:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- variable<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- variable<br>
    &ensp;&ensp;&ensp;&ensp;secrets-manager:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;key:secret-id:json-key:version-stage:version-id<br>
    &ensp;&ensp;&ensp;&ensp;git-credential-helper: no | yes<br><br>
    proxy:<br>
    &ensp;&ensp;&ensp;&ensp;upload-artifacts: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;logs: no | yes<br>
    batch:<br>
    &ensp;&ensp;&ensp;&ensp;fast-fail: false | true<br>
    &ensp;&ensp;&ensp;&ensp;# build-list:<br>
    &ensp;&ensp;&ensp;&ensp;# build-matrix:<br>
    &ensp;&ensp;&ensp;&ensp;# build-graph:<br>
    phases:<br>
    &ensp;&ensp;&ensp;&ensp;install:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;run-as: Linux-user-name<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;on-failure: ABORT | CONTINUE<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;runtime-versions:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;runtime: version<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;runtime: version<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;commands:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;finally:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;pre_build:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;run-as: Linux-user-name<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;on-failure: ABORT | CONTINUE<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;commands:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;finally:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;build:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;run-as: Linux-user-name<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;on-failure: ABORT | CONTINUE<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;commands:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;finally:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;post_build:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;run-as: Linux-user-name<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;on-failure: ABORT | CONTINUE<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;commands:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;finally:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- command<br>
    reports:<br>
    &ensp;&ensp;&ensp;&ensp;report-group-name-or-arn:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;files:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;base-directory: location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;discard-paths: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;file-format: report-format<br>
    artifacts:<br>
    &ensp;&ensp;&ensp;&ensp;files:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;name: artifact-name<br>
    &ensp;&ensp;&ensp;&ensp;discard-paths: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;base-directory: location<br>
    &ensp;&ensp;&ensp;&ensp;exclude-paths: excluded paths<br>
    &ensp;&ensp;&ensp;&ensp;enable-symlinks: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;s3-prefix: prefix<br>
    &ensp;&ensp;&ensp;&ensp;secondary-artifacts:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;artifactIdentifier:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;files:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;name: secondary-artifact-name<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;discard-paths: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;base-directory: location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;artifactIdentifier:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;files:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- location<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;discard-paths: no | yes<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;base-directory: location<br>
    cache:<br>
    &ensp;&ensp;&ensp;&ensp;paths:<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- path<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;- path<br>
- Environment Variable
  - 所有环境变量总长度不能超过`5500`个字符
  - 可用`System Manager Parameter Store`保存多个Environment Variable,每个Environment Variable不超过`4096`字符,然后在`buildsped.yml`中从`System Manager Parameter Store`中取得这些变量

## CodeCommit
- CodeCommit所需权限
  - `Git Credentials` , IAM generated, username/password, 用于HTTPS访问CodeCommit
  - `SSH Keys`, Public/Private Key Pair, 与IAM user关联, 用于SSH访问CodeCommit
  - `Access Keys`将, 用于CLI-HTTPS访问CodeCommit

## ECS
- Docker Volume可让Docker间共享存储
- 需要在Task Definition中定义Volume和Docker mount point
- 同一个Task Definition中可定义多个Contaier
- Task Placement

  |Type|Field|
  |-|-|
  |binpack|CPU|
  |~|Memory|
  |Spread|InstanceId|
  |~|attributes:ecs.availability-zone|
  |~|attributes:ecs.instance-type|
  |~|attributes:ecs.os-type|
  |~|attributes:ecs.aim-id|
  |random|~|

- `Environment` 定义
  - 需要在Task Definition中定义`Enviroment`才能传递给container
- ECS端口映射
  - 可能有多个Container都在向Host映射端口,如果都指定映射Host某端口可能会发生冲突,所以最简单的办法是Host端口指定为0,ECS会自动找到Host上没有冲突的端口进行映射<br>
  ![ecs-port](https://routescroll.github.io/ecs-port.jpeg)
- 监控ECS Container的性能
  - 要用到`X-Rays`
    1. 需要自己搞一个安装了`X-Rays`的Container Image
    2. Push到 Image Repository
    3. ECS的`Task Definition`中使用该Image部署带有X-Rays的Container
    4. 其他Container通过在`Task Definition`中设置`Port Mapping`及其他网络设置与X-Rays Container进行通信<br>
    也就是下图中间那种
    ![ecs-xrays-deamon](https://routescroll.github.io/ecs-x-ray-daemon.jpg)
- 关于 `Task Role`
  - 在 `Task Definition` 中可以为各个Container指定各自的 `Task Role` 以使各个Task可以获得各自适合的权限
  - 调用 `RunTask API` 启动新Task时也可以指定 `Task Role`

## API Gateway
- CloudWatch Error Monitor Metric
  - client-side Error Metric -> 4XX Error
  - server-side Error Metric -> 5XX Error
- 从Client发起请求Cache无效化
  - pass the HTTP header `Cache-Control: max-age=0`
  - 可以使API Gateway不去Check Cache而去后端请求最新数据
  - API Gateway Cache TTL:0~3600s, Default:300s
  Client所需权限<br>
  ![invalidate-cache](https://routescroll.github.io/invalidate-cache.jpeg)
- `RESTful API` (JSON/XML)
  - 使用`Mapping Template`将Request Payload数据映射成BackEnd需要的格式
  - 同时也会将后端的Response映射成FrontEnd所需要的格式
- `SOAP` -> Simple Object Access Protocal
- `CORS(Cross-Origin-Resource-Sharing)`
  - 需要在服务端设定是否允许`CORS`而不是请求端
- `API Gateway Latency`
  > Latency = B - A<br>
    Latency中包括API Gateway 与 Integration之间的Latency<br>
    client ---------> {request时刻A} API Gateway ---------> Integration<br>
    client <--------- {response时刻B} API Gateway <--------- Integration
- `Usage Plan`
  - 通过创建 `Usage Plan` 获得 `API Key`, 用户通过该Key访问API可以获得相应的访问量/速度
  - `API Key` 可以由 `Usage Plan` 生成, 也可以自行Import(从csv文件)
  - 要将一个 `API Key` 与一个 `Usage Plan` 关联起来, 需要调用 `createUsagePlanKey`
  - <font color=red>貌似只有REST API才能定义 `Usage Plan`</font>
- Log类型
  - `API Gateway execution log`
    - errors or execution traces
      - request or response parameter values or payloads
      - data used by Lambda authorizers
      - whether API keys are required
      - whether usage plans are enabled
  - `API Gateway access log`
    - log `who` has accessed your API and `how` the caller accessed the API
- REST API Method & Path
  ![rest-api-method-path](https://routescroll.github.io/rest-api-method-path.png)
- `Authorizer Lambda`
  > 注意 `Token Based` 里的Token也是放在Header里的, 实际考试的时候估计考点是Token Base的话可能会强调使用Token授权吧...

  |Authorizer Type|Identity Parameter|Description|
  |-|-|-|
  |Token-Based|JWT or OAuth Token|<font color=red>Token好像是要放在Header里</font>|
  |Request-Based|Header<br>Query String Parameter<br>Stage Variable<br>$context Variable<br>的任意一种或任意组合|对于WebSocket只能选择这种方式|

  ![lambda-authorizer](http://routescroll.github.io/lambda-authorizer.png)

## DynamoDB
- DynamoDB Stream
  `按修改时间顺序保存Item修改记录,Max 24小时以内记录`
- AWS Managed Policy
  `会对所有DynamoDB Resource生效, 不能指定某个DynamoDB Resource`
  
  | StreamViewType | Record Type |
  |-|-|
  |KEYS_ONLY|Key Only|
  |NEW_IMAGE|Entire New Item|
  |OLD_IMAGE|Entire Old Item|
  |NEW_AND_OLD_IMAGES|Entire New & Old Item|

- 每个Item最大`400KB`
- DynamoDB是保存`Session data`的好办法.把Session Data的保存位置从本地文件系统变成了共享位置,且快速,可扩展
- `Scan`
  - 全表访问,使用Projection或Filter获得指定值.耗费更多RCU
- `Query`
  - 只能用PartitionKey进行查询,用Filter获得指定值.耗费RCU相对少 
- `Parallel Scan with Limit Parameter` (提高Scan效率)
  - 使用 `Limit Parameter` 限制page size
  - `Parallel Scan` 的使用场景
    -  Table Size > `20GB`
    -  Table `Provisioned Read Throughput` 没有被充分利用
    -  顺序Scan过慢(Sequential Scan)
  -  > 默认情况下，Scan 操作按顺序处理数据。Amazon DynamoDB 以 1 MB 的增量向应用程序返回数据，应用程序执行其他 Scan 操作检索接下来 1 MB 的数据。扫描的表或索引越大，Scan 完成需要的时间越长。此外，一个顺序 Scan 可能并不总是能够充分利用预置读取吞吐量容量：即使 DynamoDB 跨多个物理分区分配大型表的数据，Scan 操作一次只能读取一个分区。出于这个原因，Scan 受到单个分区的最大吞吐量限制。为了解决这些问题，Scan操作可以逻辑地将表或二级索引分成多个分段，多个应用程序工作进程并行扫描这些段。每个工作进程可以是一个线程（在支持多线程的编程语言中），也可以是一个操作系统进程。要执行并行扫描，每个工作进程都会发出自己的 Scan 请求，并使用以下参数：
    Segment — 要由特定工作进程扫描的段。每个工作进程应使用不同的 Segment 值。
    TotalSegments — 并行扫描的片段总数。该值必须与应用程序将使用的工作进程数量相同。
    ![parallelscan](https://routescroll.github.io/ParallelScan.png)
  - nodejs使用例
    `Scan(TotalSegments=4, Segment=0, ...)`
    `Scan(TotalSegments=4, Segment=1, ...)`
    `Scan(TotalSegments=4, Segment=2, ...)`
    `Scan(TotalSegments=4, Segment=3, ...)`
  - CLI使用例
    `aws dynamodb scan --totalsegments x --segment y ...`
- `Scan操作对于RCU的冲击`
  ![scan](https://routescroll.github.io/GetImage.jpeg)
  - 降低Scan操作对RCU冲击的方法
    - Reduce Page Size(Query操作也适用)
      - Default Page Size:`1MB`
      - 使用`Limit parameter`
      - 这样还能在多次操作之间创造一点间隔
    - Isolate scan operations
      - 两个表交替承担关键任务 (`mission-critical`)
      - 两个表交替成为互相的Shadow Table
      - 关键任务(`mission-critical`) 表不承担Scan操作
- `TransactfWriteItems/Transactional`
  - 可以达成写操作的`idempotent` (幂等性)
  - 要么都成功,要么都失败 (`all-or-nothing operations`)
  - 一组最多`25`个Write Operation, `同Region`+`同Account`+`One/More Tables`
  - 一组Item Size总和不超过`4MB`
  - TransactWriteItems 和 BatchWriteItem 区别
    - TransactWriteItems:`all-or-nothing operations`
    - BatchWriteItem:可能有的成功有的失败
- `Conditional writes`
  - Default Write Operation
    - `Unconditional` (PutItem, UpdateItem, DeleteItem)
    - overwrites an existing item that has the specified primary key
  - A conditional write succeeds <font color=red>only if</font> the item attributes <font color=red>meet</font> one or more expected <font color=red>conditions</font>. Otherwise, it returns an error
    - eg. PutItem operation to succeed only if there is not already an item with the same primary key
    - eg. prevent an UpdateItem operation from modifying an item if one of its attributes has a certain value
- `RCU计算` 相关问题
  - MAXIMIZE the number of requests allowed per second
    - Eventually consistent, 15 RCUs, 1 KB item = <font color=red>30</font> items read/s.
    - Strongly consistent, 15 RCUs, 1 KB item = 15 items read/s.
    - Eventually consistent, 5 RCUs, 4 KB item = 10 items read/s.
    - Strongly consistent, 5 RCUs, 4 KB item = 5 items read/s.
  - The most efficient use of throughput
    - Eventually consistent, 15 RCUs, 1 KB item = 30 items/s = <font color=red>2</font> items/RCU -> 一个RCU一次可以读取<font color=red>1KB x 2RCU = 2KB</font>
    - Strongly consistent, 15 RCUs, 1 KB item = 15 items/s = 1 item per RCU -> 一个RCU一次可以读取<font color=red>1KB x 1RCU = 1KB</font>
    - Eventually consistent, 5 RCUs, 4 KB item = 10 items/s = <font color=red>2</font> items/RCU -> 一个RCU一次可以读取<font color=red>4KB x 2RCU = 8KB</font> <- <font color=blue>所以这个最高效</font>
    - Strongly consistent, 5 RCUs, 4 KB item = 5 items/s = 1 item per RCU -> 一个RCU一次可以读取<font color=red>4KB x 1RCU = 4KB</font>

## Lambda
- Lambda部署
  - `buildspec.yml`
    - 只有使用CodeBuild时buildspec.yml才起作用
    - 使用API部署Lambda时需要将代码和所有依赖都打包上传S3
  - `appspec.yml`
    - 只有使用CodeDeploy是appspec.yml才起作用
- Lambda Invoke Typoe
  - RequestResponse
    - Synchronously
  - Event
    - Asynchronously
    - Invoke失败后最多重试1次(加上第一次Invoke最多只会Invoke 2次)
    - Invoke之前Event会被先发送到一个Queue(用户不可见),之后从Queue发送到Lambda.
    - 如果Lambda处理速度赶不上Queue发送速度,Event可能会丢失
    - 偶尔Lambda会收到两次相同的Event
  - DryRun
    - Validate parameter values and verify that the user or role has permission to invoke the function.
- 可以作为Lambda Event Source Map的Service
  - `Amazon Kinesis`
  - `Amazon DynamoDB`
  - `Amazon Simple Queue Service`
- 上面以外的Service是主动Invoke Lambda(将Lambda作为Target),并且需要适当的权限才能Invoke

## CloudWatch
- Alarm
  - `High-Resolution Alarm`
    - 频度:10s / 30s / 60s整数倍
- Metric Filters
  - 用户自行创建 `Metric Filters` 可以在CloudWatch Log中搜索期望的值并数字化后作为Metric中的一维显示或用其设置Alarm
  - <font color=red>需要注意</font>: CloudWatch只会获取 `Metric Filters` 创建以后发生的Metric数据, 在此之前的Log不是新 `Metric Filter` 的查找对象
- CloudWatch Event
  - 使用`CloudWatch Event Rule`获取Service的各种变化,并传递给Targe(eg. Lambda)<br>
  ![cloudwatch-event](https://routescroll.github.io/cloudwatch-rule.png)
- CloudWatch Metric - `NetworkIn`
  - 该Metric只能看到连接用户数量, 看不到application的并发用户数量
- put-metric-data
  - `aws cloudwatch put-metric-data` 在EC2内执行,可以自定义Metric并指定频度向CloudWatch定期发送Metric Data
- 关于 `HTTP 400: ThrottlingException`
  - 调用 `PutMetricData API` 时偶尔会发生 `ThrottlingException`, 类似下图
  ![put-metric-throttle](https://routescroll.github.io/put-metric-throttle.jpeg)
  三种Troubleshooting<br>
    1. 均匀分配调用时间, 不要短时间内大量调用. 或者使用 `jitter(randamized delay)` 随机发送时刻
    2. 减少调用频率, 一次调用发送多个Metric数据 (`20 Metris & 150 data points`)
    3. Retry your call with exponential backoff and jitter
- 关于`cross-account cross-region dashboard`
  - 可以在一个账号里创建一个`CloudWatch Dashboard`, 监控其他 `Account & Region` 的Megric
  - 需要再CloudWatch里启用`cross-account cross-region`
  - 需要添加目标账户, 并得到目标账户的承认

## AppSync - GraphQL
- GraphQL与AppSync组合向用户提供 **Single API - Multi Data Source** 服务<br>
![appsync](https://routescroll.github.io/graphql.jpeg)

## Elastic Beanstalk
- 全托管
  - `Capacity Provisioning`
  - `Load Balancing`
  - `Auto Scaling`
  - `RDS Integrated (RDS Auto Scaling included)`
- 关于`启用X-Rays Daemon`
  - 可以在Beanstalk的Console中启用
  - 可以在Beanstalk的Source code中启用
  ![beanstalk](https://routescroll.github.io/beanstalk-x-rays.jpeg)

- Deploy
  - Elastic Beanstalk的Deploy要满足如下几个条件
    - 上传对象为 `Source Bundle`
    - Consist of a single ZIP file or WAR file (you can include multiple WAR files inside your ZIP file)
    - Not exceed 512 MB
    - Not include a parent folder or top-level directory (subdirectories are fine) 
    - Deploy对象为 `Worker Application` 时Source Bundle里要包含 `cron.yaml` 文件
  - Deploy图示
    - All-at-once<br>
    ![all-at-onece1](https://routescroll.github.io/all_at_once1.png)
    ![all-at-onece2](https://routescroll.github.io/all_at_once2.png)
    ![all-at-onece3](https://routescroll.github.io/all_at_once3.png)
    - Rolling<br>
    ![all-at-onece1](https://routescroll.github.io/rolling1.png)
    ![all-at-onece1](https://routescroll.github.io/rolling2-1.png)
    ![all-at-onece1](https://routescroll.github.io/rolling3.png)
    ![all-at-onece1](https://routescroll.github.io/rolling4-2.png)
    ![all-at-onece1](https://routescroll.github.io/rolling5.png)
    - Rolling additional Batch<br>
    ![all-at-onece1](https://routescroll.github.io/rolling_add1.png)
    ![all-at-onece1](https://routescroll.github.io/rolling_add2.png)
    ![all-at-onece1](https://routescroll.github.io/rolling_add6.png)
    ![all-at-onece1](https://routescroll.github.io/rolling_add3.png)
    ![all-at-onece1](https://routescroll.github.io/rolling_add4.png)
    ![all-at-onece1](https://routescroll.github.io/rolling_add5.png)
    - Immutable
    ![all-at-onece1](https://routescroll.github.io/immutable1.png)
    ![all-at-onece1](https://routescroll.github.io/immutable2-640x294.png)
    ![all-at-onece1](https://routescroll.github.io/immutaable3-640x294.png)
    ![all-at-onece1](https://routescroll.github.io/immutable4-640x294.png)
    ![all-at-onece1](https://routescroll.github.io/immutable5-640x294.png)
    ![all-at-onece1](https://routescroll.github.io/immutable6.png)

    

## Elastic Load Balancer
- `Sticky Sessions`
  - `Sticky Sessions` 只能使用户application能够重新连接到断线之前连接的EC2 instance,但是并不能保证session data仍然有效
  - 在 `Target Group` 上启用 `Sticky Sessions` 可以使用户每次连接到之前相同的Instance(避免被Load Balancer路由到其他Instance), 从而避免被路由到其他Instance发生重新验证
  - 需要客户端启用Cookies
- `x-forwarded-proto request header`
  - Server通常只能看见ELB的Public IP<br>想要看Client的实际IP<br>要在Server端配制`include the x-forwarded-for request header in the log files`
    > Server <---> ELB <---> Client
- `x-forwarded-proto request`
  - 在log中查看协议(`HTTP or HTTPS`)
- `TLS/SSL Termination/Offload`
  - TLS/SSL 卸载, 也就是解密客户端传来的加密数据
  - 卸载过程会对性能造成很大影响
  - 所以可以在Load Balancer端配置SSL证书和`TLS/SSL Termination/Offload`
  - 既保证了安全, 也减轻了后端EC2的性能压力

## X-Ray
- 用来`Tracing Application`整体流程和性能,帮助找到性能瓶颈等<br>
  ![xray](https://routescroll.github.io/xary.jpeg)<br>
  ![xray-structure](https://routescroll.github.io/x-rays-structure.jpeg)
  

## IAM
- 对于EC2可以使用`IAM Role`来访问AWS上的资源
- 对于On-Premise来说无法使用`IAM Role`,所以好的解决办法就是使用`Access Keys`,将`Access Keys`保存到本机某路径
  - `Linux/MacOS: ~/.aws/credentials `
  - `Windows: %UserProfle%\.aws\credentials`
- `Cross-Account Access`<br>
  - 不能把其他账号下的IAM user加入到本账号的IAM Group中<br>
![iam-cross-account-access](https://routescroll.github.io/iam-cross-account-access.png)
- `AWS Managed Policy` & `Customer Managed Policy` 比较
  - `Customer Managed Policy` 相对而言可以定制粒度更小的Policy
  - `AWS Managed Policy` 相对而言粒度大, 当只需要某几个权限的场合`AWS Managed Policy`通常会不适用, 要么粒度过大, 要么粒度更小


## AWS Cognito
- `User Pools` & `Identity Pools`

|Name|Description|
|-|-|
|`User Pools`|用户群体用于身份验证（身份核实）。如果使用用户群体，应用程序用户将可以通过该用户群体进行登录或通过第三方身份提供者（IdP）进行联合身份验证。|
|`Identity Pools`|身份池用于授权（访问控制）。您可以使用身份池为用户创建唯一身份并向他们授予访问其他 AWS 服务的权限。|

- `Cognito` 所支持的`Identity Pools`
  - Public providers: Amazon, Facebook, Google, Apple 
  - Amazon Cognito user pools 
  - Open ID Connect providers (identity pools) 
  - SAML identity providers (identity pools) 
  - Developer authenticated identities (identity pools) 

- `Cognito` 可以用来区分<font color=red>已验证用户</font>和<font color=red>非验证用户</font><br>从而为二者提供不同权限的Role以到达访问权限控制的目的
- `Adaptive Authentication`
  - With `Adaptive Authentication`, you can configure your user pool to block suspicious sign-ins or add second factor authentication in response to an increased risk level.<br>
  Cogtino评估每个登录请求并给出`Risk Level`(评估考虑的因素很多, 如新Device, 用户位置, IP等)<br>
  根据`Risk Level`, 用户可以有如下选择
    - Allow -> 允许登录
    - Optional MFA -> 需要登录者完成MFA登录Challenge
    - Require MFA -> 需要登录者完成MFA登录Challenge, 没有配置MFA则会被Block
    - Block -> 直接Block
- `认证 & 授权 流程`
  ![cognito-userpool-id-pool](https://routescroll.github.io/cogtino-userpool-idpool-exchange.png)
- `Cogtino提供登录页面`
  > When you create a user pool in Amazon Cognito and then configure a domain for it, Amazon Cognito automatically provisions a hosted web UI to let you add sign-up and sign-in pages to your app. You can add a custom logo or customize the CSS for the hosted web UI.
  1. 在Cognito中创建 `user pool`
  2. 为该 `user pool` 配置一个domain
  3. Cognito会为你的app自动提供一个Web UI供用户注册登录. 用户可以向该页面添加自己的CSS或者logo

## Step Function
- `Fields Filter` , 用于从InputJSON中选择特定项目使用
  - `InputPath` -> 用于选择输入JSON特定项目
  - `OutputPath` -> 用于选择传往下个状态JSON特定项目(从Input中选择)
  - `ResultPath` -> 用于选择最终输出JSON特定项目(从Input中选择)
      > 例如,原始Input JSON:
    {<br>
    &ensp;&ensp;&ensp;&ensp;"comment": "An input comment.",<br>
    &ensp;&ensp;&ensp;&ensp;"data": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"val1": 23,<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"val2": 17<br>
    &ensp;&ensp;&ensp;&ensp;},<br>
    &ensp;&ensp;&ensp;&ensp;"extra": "foo",<br>
    &ensp;&ensp;&ensp;&ensp;"lambda": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"who": "AWS Step Functions"<br>
    &ensp;&ensp;&ensp;&ensp;}<br>
    }<br><br>
    Step Function定义<br>
    {<br>
    &ensp;&ensp;&ensp;&ensp;"Comment": "A Catch example of the Amazon States Language using an AWS Lambda function",<br>
    &ensp;&ensp;&ensp;&ensp;"StartAt": "CreateAccount",<br>
    &ensp;&ensp;&ensp;&ensp;"States": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"CreateAccount": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Type": "Task",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Resource": "arn:aws:lambda:us-east-1:123456789012:function:FailFunction",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"InputPath": "\$.lambda",</font><font color=blue><-真正输入到Function中的只有lambda项目</font><br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"OutputPath": "\$.data",</font><font color=blue><-传递个下一个状态Function的只有data项目</font><br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ResultPath": "\$.data.lambdaresult",</font><font color=blue><-输出结果会放到data.lambdaresult下面,输出的内容只有data项目</font><br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Catch": [ {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"ErrorEquals": ["CustomError"],<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Next": "CustomErrorFallback"<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ResultPath": "$.data.error",</font></font><font color=blue><-发生异常时结果会放到data.error下面</font><br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}, {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"ErrorEquals": ["States.TaskFailed"],<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Next": "ReservedTypeFallback"<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}, {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"ErrorEquals": ["States.ALL"],<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Next": "CatchAllFallback"<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;} ],<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"End": true<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"CustomErrorFallback": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Type": "Pass",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Result": "This is a fallback from a custom Lambda function exception",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"End": true<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"ReservedTypeFallback": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Type": "Pass",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Result": "This is a fallback from a reserved error code",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"End": true<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"CatchAllFallback": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Type": "Pass",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Result": "This is a fallback from any error code",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"End": true<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}<br>
    &ensp;&ensp;&ensp;&ensp;}<br>
    }<br>
    - 回顾 - 原始输入JSON
      >{<br>
    &ensp;&ensp;&ensp;&ensp;"comment": "An input comment.",<br>
    &ensp;&ensp;&ensp;&ensp;"data": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"val1": 23,<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"val2": 17<br>
    &ensp;&ensp;&ensp;&ensp;},<br>
    &ensp;&ensp;&ensp;&ensp;"extra": "foo",<br>
    &ensp;&ensp;&ensp;&ensp;"lambda": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"who": "AWS Step Functions"<br>
    &ensp;&ensp;&ensp;&ensp;}<br>
    }<br>
    - 过滤后Input:
      >{<br>
    &ensp;&ensp;&ensp;&ensp;<font color=blue>{"who": "AWS Step Functions"}</font><br>
    }<br>
    - 过滤后Output:
      >{<br>
    <font color=green>&ensp;&ensp;&ensp;&ensp;"val1": 23,<br>
    &ensp;&ensp;&ensp;&ensp;"val2": 17</font><br>
    }<br>
    - 过滤后Result:
      >{<br>
    <font color=fuchsia>&ensp;&ensp;&ensp;&ensp;"val1": 23,<br>
    &ensp;&ensp;&ensp;&ensp;"val2": 17</font><br>
    }<br>
    - 过滤后Exception:
      >{<br>
    <font color=orange>&ensp;&ensp;&ensp;&ensp;"val1": 23,<br>
    &ensp;&ensp;&ensp;&ensp;"val2": 17<br>
    &ensp;&ensp;&ensp;&ensp;"error": error message</font><br>
    }<br>

- `Parameters` -> 从Input中选择某些项目重新组合成新的Input内容
  > {<br>
&ensp;&ensp;&ensp;&ensp;"comment": "Example for Parameters.",<br>
&ensp;&ensp;&ensp;&ensp;"product": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"details": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"color": "blue",<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"size": "small",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"material": "cotton"<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"availability": "in stock",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"sku": "2317",<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"cost": "$23"<br>
&ensp;&ensp;&ensp;&ensp;}<br>
}<br>
使用Parameters重新映射成新Input<br>
"Parameters": {<br>
&ensp;&ensp;&ensp;&ensp;"comment": "Selecting what I care about.",<br>
&ensp;&ensp;&ensp;&ensp;"MyDetails": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"size.\$": "\$.product.details.size",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"exists.\$": "\$.product.availability",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"StaticValue": "foo"<br>
&ensp;&ensp;&ensp;&ensp;}<br>
},<br>
映射后的新Input<br>
{<br>
&ensp;&ensp;&ensp;&ensp;"comment": "Selecting what I care about.",<br>
&ensp;&ensp;&ensp;&ensp;"MyDetails": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"size": "small",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"exists": "in stock",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"StaticValue": "foo"<br>
&ensp;&ensp;&ensp;&ensp;}<br>
}<br>

- `ResultSelector` 从输出结果中选择特定项目,应用于`ResultPath`之前
  > 原始Result<br>
{<br>
&ensp;&ensp;&ensp;&ensp;<font color=red>"resourceType": "elasticmapreduce",</font><br>
&ensp;&ensp;&ensp;&ensp;"resource": "createCluster.sync",<br>
&ensp;&ensp;&ensp;&ensp;"output": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"SdkHttpMetadata": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"HttpHeaders": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Content-Length": "1112",<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Content-Type": "application/x-amz-JSON-1.1",<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Date": "Mon, 25 Nov 2019 19:41:29 GMT",<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"x-amzn-RequestId": "1234-5678-9012"<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"HttpStatusCode": 200<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"SdkResponseMetadata": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"RequestId": "1234-5678-9012"<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ClusterId": "AKIAIOSFODNN7EXAMPLE"</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}<br>
}<br>
应用`ResultSelector`<br>
"Create Cluster": {<br>
&ensp;&ensp;&ensp;&ensp;"Type": "Task",<br>
&ensp;&ensp;&ensp;&ensp;"Resource": "arn:aws:states:::elasticmapreduce:createCluster.sync",<br>
&ensp;&ensp;&ensp;&ensp;"Parameters": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=silver>snowsome parameters></font><br>
&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;"ResultSelector": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ClusterId.$": "$.output.ClusterId",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ResourceType.$": "$.resourceType"</font><br>
&ensp;&ensp;&ensp;&ensp;},<br>
&ensp;&ensp;&ensp;&ensp;"ResultPath": "$.EMROutput",<br>
&ensp;&ensp;&ensp;&ensp;"Next": "Next Step"<br>
}<br>
最终 Result<br>
{<br>
<font color=silver>"OtherDataFromInput": {},</font><br>
&ensp;&ensp;&ensp;&ensp;"EMROutput": {<br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ResourceType": "elasticmapreduce",</font><br>
&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"ClusterId": "AKIAIOSFODNN7EXAMPLE"</font><br>
&ensp;&ensp;&ensp;&ensp;}<br>
}<br>

- State的Type

  |Type|Description|Usecase|
  |-|-|-|
  |Pass|passes its input to its output, without performing work.<br>Pass states are useful|when constructing and debugging state machines.|
  |Task|All work in your state machine is done by tasks|A task performs work by using <br>`an activity`<br>`or an AWS Lambda function`<br>`or by passing parameters to the API actions of other services`|
  |Choice|adds conditional logic to a state machine.||
  |Wait|delays the state machine from continuing for a specified time|a relative time, specified in seconds from when the state begins<br>or an absolute end time, specified as a timestamp.|
  |Succeed|tops an execution successfully|Succeed state is a useful target for Choice state branches that don't do anything but stop the execution|
  |Fail|stops the execution of the state machine and marks it as a failure, unless it is caught by a Catch block.||
  |Parallel|can be used to add separate branches of execution in your state machine.||
  |Map|run a set of workflow steps for each item in a dataset|The Map state's iterations run in parallel, which makes it possible to process a dataset quickly<br>executes the same steps for multiple entries of an array in the state input.|

##  AWS Systems Manager Parameter Store
- `Systems Manager Parameter Store` 与 `Key Management Store` 结合使用保存敏感文本数据(比如证书).
- `Parameter` 使用 `KMS` 加密/解密文本.
- 其他Application/Service(比如Lambda)可以通过引用`Parameter/KMS`中的内容实现代码中无明文使用敏感数据
- `Systems Manager Parameter Store` 不会自动轮换加密证书之类的,需要application自己去做轮换.
- [参考:Difference between Parameter Store & Secrets Manager](https://medium.com/awesome-cloud/aws-difference-between-secrets-manager-and-parameter-store-systems-manager-f02686604eae)

## AWS KMS
- `AWS Managed CMKs`
  - AWS负责管理Keys, 当与其他AWS Service交互时代表用户使用这些Keys.
- `Customer managed CMSs`
  - 用户负责管理Keys, 用户可以管理这些Keys的各种策略
- `Data Keys`通常用来加密解密数据, 可以由KMS CMKs生成`Data Keys`. 但是AWS不负责保存这些`Data Keys`, 用户需要自己负责`Data Keys`安全.
- 用户可以直接使用KMS CMKs去加密数据, 避免Data Keys保管在AWS之外造成安全问题
- 获取 `Data Key` 的API
  - `GenerateDataKey` -> 从KMS获取Data Key(Plain Text), 该Data Key可用于加密数据
  - `GenerateDataKeyWithoutPlainText` -> 从KMS获取Data Key(Encrypted Text), 用于加密数据之前需要再调用KMS API解密该Data Key
  - KMS只能直接加密不超过4096字节的数据, 所以不适于用来加密大型数据, 只能用于加密Key这类的小数据<br><br>
    用户指定一个对称加密KMS Key, 用于加密生成的`Data Key`
    ![generate-data-key](https://routescroll.github.io/generate-data-key.jpeg)

    使用Plaintext Data Key加密数据
    ![generate-data-key](https://routescroll.github.io/encrypt-data.jpeg)

## S3
- `Static WebSite` 启用
  - `Disable Block Public Access`
  - 新增Bucket Polic, 赋予所有人`s3:GetObject`权限
  - 上传一个index Document, 比如`index.html`
- `Notifications Events`
  - New object created events
    - 新增Object
      - s3:ObjectCreated:*
        - 包括Put, Post, Copy等
  - Object removal events
  - Restore object events
  - Reduced Redundancy Storage (RRS) object lost events
  - Replication events
- 访问权限控制
  - S3可以针对个别用户进行访问权限设置, 但是当有大量用户的时候, S3的访问权限控制会变得十分麻烦, 每当新增或删除用户时都要重新设置访问权限控制策略
- Read/Write 速度限制
  > 可以通过设置多个Prefix把文件分散开, 充分利用每个Prefix的速率限制, 避免超过R/W的限制引发错误
  - Prefix数量无限制
  - Write
    - 3500/Prefix
  - Read
    - 5500/Prefix
- 关于 `Serside Encryption`
  - 如果Bucket Policy设置了 `强制上传加密`, 那么客户端 `PutObject` API的Header中一定要加入 `x-amz-server-side-encryption` 选项才能合规
  - 在这中情形下, 只在Bucket上设定 `Serside Encryption` 并不足以合规(符合Policy)
  - Polic例
    > {<br>
    &ensp;&ensp;"Version": "2012-10-17",<br>
    &ensp;&ensp;"Id": "PutObjectPolicy",<br>
    &ensp;&ensp;"Statement": [<br>
    &ensp;&ensp;&ensp;&ensp;{<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Sid": "DenyIncorrectEncryptionHeader",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Effect": "Deny",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Principal": "*",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Action": "s3:PutObject",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Resource": "arn:aws:s3:::awsexamplebucket1/*",<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"Condition": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;"StringNotEquals": {<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;<font color=red>"s3:x-amz-server-side-encryption": "AES256"</font><br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}<br>
    &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;}<br>
    &ensp;&ensp;&ensp;&ensp;}<br>
    &ensp;&ensp;]<br>
  }<br>

## Elastic Cache
- `Redis`
  - `Redis` 在`Single Shard & Single Node`模式下一旦Node完蛋所有数据全毁
    如果运行`Redis Engines`, `Single Shard` 中有多个 `Replica Node`. 即使一个Node挂掉别的Node也可以做Failover<br><br>
    <font color=red>Redis Single Shard & Multi Nodes</font> - 一个Shard即保存了所有数据, Shard中一个`Primary Node`(实际所有数据都在`Primary Node`中), 另外最多可以有<font color=red>5</font>个read-only replica node.
    因为Primary和Replica之间复制时有Latency, 所以当`Primary Node`挂掉时replica node可能会有数据不同步, 发生数据丢失
    ![redis-node-replica](https://routescroll.github.io/redis-single.jpeg)
    
    <font color=red>Redis Multi Shard & Multi Nodes</font>
    ![redis-node-replica](https://routescroll.github.io/redis-cluster.jpeg)
    
    <font color=red>Memcached Multi Nodes</font> - 所有数据分布在各个Node中, 一个Node挂掉所有数据完整性会被破坏
    ![redis-node-replica](https://routescroll.github.io/memcached.jpeg)

## CloudFormation
- `CloudFormation Depolyment`有时会出现删除某Reource失败, 原因可能是某Resource与其他非当前Stack创建的Resource有关联, 当前Stack无法删除非自己创建的Resource, 也无法删除与之关联的自己创建的Resource
  - 解决方式: 修改CloudFormation Template, retain无法删除的resource, 等Deployment结束后手动删除该资源
  - CLI删除Stack时碰见这种情况可以这么做:<br>
    `aws cloudformation delete-stack --stack-name my-stack --retain-resources xxxx`
- template.yml CLI Deploy
  > sam package / sam deploy 同样适用
  1. run `aws cloudformation package --template-file /path_to_template/template.json --s3-bucket bucket-name --output-template-file packaged-template.json`
  2. run `aws cloudformation deploy xxxxx`

## Resource Group
- 可以把想要监控的Resource聚合成一个组, 显示到Management Console首页, 方便管理, 不需要在各个Service之间跳来跳去
- 聚合类型分 `Tag Baase` 和 `CloudFormation stack Based`

## EC2
- `Instance Profile`
  - 通过Console创建EC2所用的Role时, Console实际上会创建一个和Role同名的 `Instance Profile`.
  - 在Console中选择和EC2 Instance关联的Role时实际上选择的是 `Instance Profile`
  - Console不会创建一个没有与EC2关联的Instance Profile
- 用CLI给EC2实例赋予Instance Profile
  1. `aws iam create-instance-profile --instance-profile-name EXAMPLEPROFILENAME`
  2. `aws iam add-role-to-instance-profile --instance-profile-name EXAMPLEPROFILENAME --role-name EXAMPLEROLENAME `
  3. `aws ec2 associate-iam-instance-profile --iam-instance-profile Arn=EXAMPLEARNNAME,Name=EXAMPLEPROFILENAME --instance-id i-012345678910abcde`

## CloudFront
- `Lambda@Edge`<br>
![lambda@edge](https://routescroll.github.io/cloudfront-events-that-trigger-lambda-functions.png)
  - CloudFront events that can trigger a Lambda@Edge function<br>
    - `Viewer request`<br>
    The function executes when CloudFront receives a request from a viewer, before it checks to see whether the requested object is in the CloudFront cache.<br>
    - `Origin request`<br>
    The function executes only when CloudFront forwards a request to your origin. When the requested object is in the CloudFront cache, the function doesn't execute.<br>
    - `Origin response`<br>
    The function executes after CloudFront receives a response from the origin and before it caches the object in the response. Note that the function executes even if an error is returned from the origin.<br>
    The function <font color=red>doesn't execute</font> in the following cases:<br>
      - When the requested file is in the CloudFront cache and is not expired.<br>
      - When the response is generated from a function that was triggered by an `origin request` event.<br>
    - `Viewer response`<br>
    The function executes before returning the requested file to the viewer. Note that the function executes regardless of whether the file is already in the CloudFront cache.<br>
    The function <font color=red>doesn't execute</font> in the following cases:<br>
      - When the origin returns an HTTP status code of 400 or higher.<br>
      - When a custom error page is returned.<br>
      - When the response is generated from a function that was triggered by a `viewer request` event.<br>
      - When CloudFront automatically redirects an HTTP request to HTTPS (when the value of Viewer protocol policy is Redirect HTTP to HTTPS).<br>
- `HTTPS & SSL/TLS`
  - Origin <===`Origin Protocol Policy`===> CloudFront <===`Viewer Protocol Policy`===> Viewer
  - `Origin Access Identity (OAI)` 是用来保护S3上的Objects的
  
## Kinesis
- `Kinesis Client Library` 运行在Consumer EC2集群里, 负责处理数据
  ![kinesis-flow.png](https://routescroll.github.io/kinesis-flow.png)

# 个别题型
### A developer is creating an Auto Scaling group of Amazon EC2 instances. The developer needs to publish a custom metric to Amazon CloudWatch. Which method would be the MOST secure way to authenticate a CloudWatch PUT request? 

- Create an IAM role with the PutMetricData permission and create a new Auto Scaling launch configuration to launch instances using that role （正确）
- Create an IAM role with the PutMetricData permission and modify the Amazon EC2 instances to use that role （错误）
- > INCORRECT: "Create an IAM role with the PutMetricData permission and modify the Amazon EC2 instances to use that role" is incorrect as you <font color=red>should create a new launch configuration</font> for the Auto Scaling group <font color=red>rather than updating the instances manually</font>.

### 问题 23：要回看教程 DynamoDB

跳过 

A gaming company is building an application to track the scores for their games using an Amazon DynamoDB table. Each item in the table is identified by a partition key 
(user_id) and a sort key (game_name). The table also includes the attribute “TopScore”. The table design is shown below: 

Primary key* 
Partition key 
user id 
OAdd sort key 
game_name 
String 
String 
A Developer has been asked to write a leaderboard application to display the highest achieved scores for each game (game_name), based on the score identified in the “TopScore” attribute. 

What process will allow the Developer to extract results MOST efficiently from the DynamoDB table? 

Use a DynamoDB scan operation to retrieve the scores for “game_name” using the “TopScore” attribute, and order the results based on the score attribute 

Create a global secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute 

（正确） 

Create a local secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute 

Create a global secondary index with a partition key of “user_id” and a sort key of “game_name” and get the results based on the “TopScore” attribute 

注解 

In an Amazon DynamoDB table, the primary key that uniquely identifies each item in the table can be composed not only of a partition key, but also of a sort key. 

Well-designed sort keys have two key benefits: 

- They gather related information together in one place where it can be queried efficiently. Careful design of the sort key lets you retrieve commonly needed groups of related items using range queries with operators such as begins_with, between, >, <, and so on. 

- Composite sort keys let you define hierarchical (one-to-many) relationships in your data that you can query at any level of the hierarchy. 

To speed up queries on non-key attributes, you can create a global secondary index. A global secondary index contains a selection of attributes from the base table, but they are organized by a primary key that is different from that of the table. The index key does not need to have any of the key attributes from the table. It doesn't even need to have the same key schema as a table. 

For this scenario we need to identify the top achieved score for each game. The most efficient way to do this is to create a global secondary index using “game_name” as the partition key and “TopScore” as the sort key. We can then efficiently query the global secondary index to find the top achieved score for each game. 

Add index 
Primary key* 
Index name* 
Projected attributes 
Partition key 
game_name 
O Add sort key 
TopScore 
game_name-TopScore-index 
All 
n Create as Local Secondary Index 
Cancel 
String 
String 
x 
o 
O 
O 
O 
Add index 
CORRECT: "Create a global secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute" is the correct answer. 

INCORRECT: "Create a local secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute" is incorrect. With a local secondary index you can have a different sort key but the partition key is the same. 

INCORRECT: "Use a DynamoDB scan operation to retrieve the scores for “game_name” using the “TopScore” attribute, and order the results based on the score attribute" is incorrect. This would be inefficient as it scans the whole table. First, we should create a global secondary index, and then use a query to efficiently retrieve the data. 

INCORRECT: "Create a global secondary index with a partition key of “user_id” and a sort key of “game_name” and get the results based on the score attribute" is incorrect as with a global secondary index you have a different partition key and sort key. Also, we don’t need “user_id”, we need “game_name” and “TopScore”. 

References: 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/amazon-dynamodb/ 

### 问题 54：要回看教程 DynamoDB

跳过 

A Developer has added a Global Secondary Index (GSI) to an existing Amazon DynamoDB table. The GSI is used mainly for read operations whereas the primary table is extremely write-intensive. Recently, the Developer has noticed throttling occurring under heavy write activity on the primary table. However, the write capacity units on the primary table are not fully utilized. 

What is the best explanation for why the writes are being throttled on the primary table? 

The write capacity units on the GSI are under provisioned 

（正确） 

There are insufficient write capacity units on the primary table 

The Developer should have added an LSI instead of a GSI 

There are insufficient read capacity units on the primary table 

注解 

Some applications might need to perform many kinds of queries, using a variety of different attributes as query criteria. To support these requirements, you can create one or more global secondary indexes and issue Query requests against these indexes in Amazon DynamoDB. 

When items from a primary table are written to the GSI they consume write capacity units. It is essential to ensure the GSI has sufficient WCUs (typically, at least as many as the primary table). If writes are throttled on the GSI, the main table will be throttled (even if there’s enough WCUs on the main table). LSIs do not cause any special throttling considerations. 

In this scenario, it is likely that the Developer assumed that the GSI would need fewer WCUs as it is more read-intensive and neglected to factor in the WCUs required for writing data into the GSI. Therefore, the most likely explanation is that the write capacity units on the GSI are under provisioned 

CORRECT: "The write capacity units on the GSI are under provisioned" is the correct answer. 

INCORRECT: "There are insufficient read capacity units on the primary table" is incorrect as the table is being throttled due to writes, not reads. 

INCORRECT: "The Developer should have added an LSI instead of a GSI" is incorrect as a GSI has specific advantages and there was likely good reason for adding a GSI. Also, you cannot add an LSI to an existing table. 

INCORRECT: "There are insufficient write capacity units on the primary table" is incorrect as the question states that the WCUs are underutilized. 

References: 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html 

### 问题 49：要回看教程 X-Rays

跳过 

An application is instrumented to generate traces using AWS X-Ray and generates a large amount of trace data. A Developer would like to use filter expressions to filter the results to specific key-value pairs added to custom subsegments. 

How should the Developer add the key-value pairs to the custom subsegments? 

Add metadata to the custom subsegments 

Setup sampling for the custom subsegments 

Add the key-value pairs to the Trace ID 

Add annotations to the custom subsegments 

（正确） 

注解 

You can record additional information about requests, the environment, or your application with annotations and metadata. You can add annotations and metadata to the segments that the X-Ray SDK creates, or to custom subsegments that you create. 

Annotations are key-value pairs with string, number, or Boolean values. Annotations are indexed for use with filter expressions. Use annotations to record data that you want to use to group traces in the console, or when calling the GetTraceSummaries API. 

Metadata are key-value pairs that can have values of any type, including objects and lists, but are not indexed for use with filter expressions. Use metadata to record additional data that you want stored in the trace but don't need to use with search. 

Annotations can be used with filter expressions, so this is the best solution for this requirement. The Developer can add annotations to the custom subsegments and will then be able to use filter expressions to filter the results in AWS X-Ray. 

CORRECT: "Add annotations to the custom subsegments" is the correct answer. 

INCORRECT: "Add metadata to the custom subsegments" is incorrect as though you can add metadata to custom subsegments it is not indexed and cannot be used with filters. 

INCORRECT: "Add the key-value pairs to the Trace ID" is incorrect as this is not something you can do. 

INCORRECT: "Setup sampling for the custom subsegments " is incorrect as this is a mechanism used by X-Ray to send only statistically significant data samples to the API. 

References: 

https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-java-segment.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/aws-developer-tools/ 

### 问题 53：要回看教程 X-Rays

跳过 

An application has been instrumented to use the AWS X-Ray SDK to collect data about the requests the application serves. The Developer has set the user field on segments to a string that identifies the user who sent the request. 

How can the Developer search for segments associated with specific users? 

Use a filter expression to search for the user field in the segment annotations 

By using the GetTraceGraph API with a filter expression 

Use a filter expression to search for the user field in the segment metadata 

By using the GetTraceSummaries API with a filter expression 

（正确） 

注解 

A segment document conveys information about a segment to X-Ray. A segment document can be up to 64 kB and contain a whole segment with subsegments, a fragment of a segment that indicates that a request is in progress, or a single subsegment that is sent separately. You can send segment documents directly to X-Ray by using the PutTraceSegments API. 

Example minimally complete segment: 

{ 

"name" : "example.com", 

"id" : "70de5b6f19ff9a0a", 

"start_time" : 1.478293361271E9, 

"trace_id" : "1-581cf771-a006649127e371903a2de979", 

"end_time" : 1.478293361449E9 

} 

 
 

A subset of segment fields are indexed by X-Ray for use with filter expressions. For example, if you set the user field on a segment to a unique identifier, you can search for segments associated with specific users in the X-Ray console or by using the GetTraceSummaries API. 

CORRECT: "By using the GetTraceSummaries API with a filter expression" is the correct answer. 

INCORRECT: "By using the GetTraceGraph API with a filter expression" is incorrect as this API action retrieves a service graph for one or more specific trace IDs. 

INCORRECT: "Use a filter expression to search for the user field in the segment metadata" is incorrect as the user field is not part of the segment metadata and metadata is not is not indexed for search. 

INCORRECT: "Use a filter expression to search for the user field in the segment annotations" is incorrect as the user field is not part of the segment annotations. 

References: 

https://docs.aws.amazon.com/xray/latest/devguide/xray-api-segmentdocuments.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/aws-developer-tools/ 