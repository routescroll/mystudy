# Explanation

## Aurora
- Aurora AutoScaling策略：基于Average connections(平均连接数)
  > Average connections of Aurora Replicas which will create a policy based on the average number of connections to Aurora Replicas.


## CodeDeploy
-
  |TST Linear|TST Canary|TST All-at-once|Service|DPT In-place|DPT Blue/green|
  |-|-|-|-|-|-|-|
  |-|-|-|EC2/On-Premises|◎|◎|
  |◎|◎|◎|Lambda|-|◎|
  |◎|◎|◎|ECS|-|◎|
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

- 通过创建多个Deployment Groupt来实现多Stage分别Deploy

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
  | Type | Field |
  |-|-|
  |binpack|CPU|
  ||Memory|
  |Spread|InstanceId|
  ||attributes:ecs.availability-zone|
  ||attributes:ecs.instance-type|
  ||attributes:ecs.os-type|
  ||attributes:ecs.aim-id|
  |random|-|
- `Environment` 定义
  - 需要在Task Definition中定义`Enviroment`才能传递给container
- ECS端口映射
  - 可能有多个Container都在向Host映射端口,如果都指定映射Host某端口可能会发生冲突,所以最简单的办法是Host端口指定为0,ECS会自动找到Host上没有冲突的端口进行映射<br>
  ![ecs-port](https://routescroll.github.io/ecs-port.jpeg)

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
    TotalSegments — 并行扫描的片段总数。该值必须与应用程序将使用的工作进程数量相同。<br>
    ![parallelscan](https://routescroll.github.io/ParallelScan.png)
  - nodejs使用例
    `Scan(TotalSegments=4, Segment=0, ...)`
    `Scan(TotalSegments=4, Segment=1, ...)`
    `Scan(TotalSegments=4, Segment=2, ...)`
    `Scan(TotalSegments=4, Segment=3, ...)`
  - CLI使用例
    `aws dynamodb scan --totalsegments x --segment y ...`
- `Scan操作对于RCU的冲击`<br>
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
- put-metric-data
  - `aws cloudwatch put-metric-data` 在EC2内执行,可以自定义Metric并指定频度向CloudWatch定期发送Metric Data
- Metric Filters
  - 用户自行创建 `Metric Filters` 可以在CloudWatch Log中搜索期望的值并数字化后作为Metric中的一维显示或用其设置Alarm
- CloudWatch Event
  - 使用`CloudWatch Event Rule`获取Service的各种变化,并传递给Targe(eg. Lambda)<br>
  ![cloudwatch-event](https://routescroll.github.io/cloudwatch-rule.png)

## AppSync - GraphQL
- GraphQL与AppSync组合向用户提供 **Single API - Multi Data Source** 服务<br>
![appsync](https://routescroll.github.io/graphql.jpeg)

## Elastic Beanstalk
- 全托管
  - `Capacity Provisioning`
  - `Load Balancing`
  - `Auto Scaling`
  - `RDS Integrated (RDS Auto Scaling included)`

- Deploy
  - Elastic Beanstalk的Deploy要满足如下几个条件
    - 上传对象为 `Source Bundle`
    - Consist of a single ZIP file or WAR file (you can include multiple WAR files inside your ZIP file)
    - Not exceed 512 MB
    - Not include a parent folder or top-level directory (subdirectories are fine) 
    - Deploy对象为 `Worker Application` 时Source Bundle里要包含 `cron.yaml` 文件

## Elastic Load Balancer
- `Sticky Sessions`
  - `Sticky Sessions` 只能使用户application能够重新连接到断线之前连接的EC2 instance,但是并不能保证session data仍然有效
- `x-forwarded-proto request header`
  - Server通常只能看见ELB的Public IP<br>想要看Client的实际IP<br>要在Server端配制`include the x-forwarded-for request header in the log files`
    > Server <---> ELB <---> Client
- `x-forwarded-proto request`
  - 在log中查看协议(`HTTP or HTTPS`)

## X-Ray
- 用来`Tracing Application`整体流程和性能,帮助找到性能瓶颈等<br>
  ![xray](https://routescroll.github.io/xary.jpeg)
  

## IAM
- 对于EC2可以使用`IAM Role`来访问AWS上的资源
- 对于On-Premise来说无法使用`IAM Role`,所以好的解决办法就是使用`Access Keys`,将`Access Keys`保存到本机某路径
  - `Linux/MacOS: ~/.aws/credentials `
  - `Windows: %UserProfle%\.aws\credentials`
- <font color=red>Amazon Cognito?????????????</font>

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
    }<br>
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

##  AWS Systems Manager Parameter Store
- `Systems Manager Parameter Store` 与 `Key Management Store` 结合使用保存敏感文本数据(比如证书).
- `Parameter` 使用 `KMS` 加密/解密文本.
- 其他Application/Service(比如Lambda)可以通过引用`Parameter/KMS`中的内容实现代码中无明文使用敏感数据
- `Systems Manager Parameter Store` 不会自动轮换加密证书之类的,需要application自己去做轮换.
- [参考:Difference between Parameter Store & Secrets Manager](https://medium.com/awesome-cloud/aws-difference-between-secrets-manager-and-parameter-store-systems-manager-f02686604eae)
