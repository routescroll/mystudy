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
    ```yaml
    files:
      - source: source-file-location-1
        destination: destination-file-location-1
      - source: source-file-location-2
        destination: destination-file-location-2
    file_exists_behavior: DISALLOW|OVERWRITE|RETAIN
    ```

  - `permission section (EC2/On-Premises only)`
    ```yaml
    permissions:
      - object: object-specification
        pattern: pattern-specification
        except: exception-specification
        owner: owner-account-name
        group: group-name
        mode: mode-specification
        acls:
          - acls-specification
        context:
          user: user-specification
          type: type-specification
          range: range-specification
        type:
          - object-type
    ```

  - `resources section (ECS & Lambda only)`
    ```yaml
    resources:
      - name-of-function-to-deploy:
          type: "AWS::Lambda::Function"
          properties:
            name: name-of-lambda-function-to-deploy
            alias: alias-of-lambda-function-to-deploy
            currentversion: version-of-the-lambda-function-traffic-currently-points-to
            targetversion: version-of-the-lambda-function-to-shift-traffic-to
    ```

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
- `Predefined Lambda Deployment Configuration (Trrafic Shifting)`
  
  |Deployment configuration (Trrafic Shifting)|
  |-|
  |CodeDeployDefault.LambdaCanary10Percent5Minutes|
  |CodeDeployDefault.LambdaCanary10Percent10Minutes|
  |CodeDeployDefault.LambdaCanary10Percent15Minutes|
  |CodeDeployDefault.LambdaCanary10Percent30Minutes|
  |CodeDeployDefault.LambdaLinear10PercentEvery1Minute|
  |CodeDeployDefault.LambdaLinear10PercentEvery2Minutes|
  |CodeDeployDefault.LambdaLinear10PercentEvery3Minutes|
  |CodeDeployDefault.LambdaLinear10PercentEvery10Minutes|
  |CodeDeployDefault.LambdaAllAtOnce|

## CodeBuild
- buildspec **<font color=red >post_build</font>**
  - 可选项
  - 在Build Phase后执行,用于发布Build后的jar/war或Docker Image等
  - **<font color=red >finall block</font>** 中的命令无论 **<font color=red >command block</font>** 的执行结果是否成功都会执行
- buildspec sample<br>
  ```yaml
    version: 0.2

    run-as: Linux-user-name
  
    env:
      shell: shell-tag
      variables:
        key: "value"
        key: "value"
      parameter-store:
        key: "value"
        key: "value"
      exported-variables:
        - variable
        - variable
      secrets-manager:
        key:secret-id:json-key:version-stage:version-id
      git-credential-helper: no | yes

    proxy:
      upload-artifacts: no | yes
      logs: no | yes

    batch:
      fast-fail: false | true
      # build-list:
      # build-matrix:
      # build-graph:

    phases:
      install:
        run-as: Linux-user-name
        on-failure: ABORT | CONTINUE
        runtime-versions:
          runtime: version
          runtime: version
        commands:
          - command
          - command
        finally:
          - command
          - command

      pre_build:
        run-as: Linux-user-name
        on-failure: ABORT | CONTINUE
        commands:
          - command
          - command
        finally:
          - command
          - command

      build:
        run-as: Linux-user-name
        on-failure: ABORT | CONTINUE
        commands:
          - command
          - command
        finally:
          - command
          - command

      post_build:
        run-as: Linux-user-name
        on-failure: ABORT | CONTINUE
        commands:
          - command
          - command

      finally:
        - command
        - command

    reports:
      report-group-name-or-arn:
        files:
          - location
          - location
        base-directory: location
        discard-paths: no | yes
        file-format: report-format

    artifacts:
      files:
        - location
        - location
      name: artifact-name
      discard-paths: no | yes
      base-directory: location
      exclude-paths: excluded paths
      enable-symlinks: no | yes
      s3-prefix: prefix
      secondary-artifacts:
        artifactIdentifier:
          files:
            - location
            - location
          name: secondary-artifact-name
          discard-paths: no | yes
          base-directory: location
        artifactIdentifier:
          files:
            - location
            - location
          discard-paths: no | yes
          base-directory: location

    cache:
      paths:
        - path
        - path
  ```
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

- Placement Contraint
  1. **distinctInstance**
  2. **memberOf** (定义方法: <font color=red>Cluster Query Language</font>)
  
      |Format|Sample|
      |-|-|
      |subject operator [argument]|attribute:ecs.instance-type == t2.small|
      ||attribute:ecs.availability-zone in [us-east-1a, us-east-1b]|
      ||task:group == service:production|
      ||not(task:group == database)|
      ||runningTasksCount == 1|
      ||ec2InstanceId in ['i-abcd1234', 'i-wxyx7890']|
      ||attribute:ecs.instance-type =~ g2.* and attribute:ecs.availability-zone != us-east-1d|

   3. Cluster query language

      |Operator|Description|
      |-|-|
      |==, equals|String equality|
      |!=, not_equals|String inequality|
      |>, greater_than|Greater than|
      |>=, greater_than_equal|Greater than or equal to|
      |<, less_than|Less than|
      |<=, less_than_equal|Less than or equal to|
      |exists|Subject exists|
      |!exists, not_exists|Subject doesn't exist|
      |in|Value in argument list|
      |!in, not_in|Value not in argument list|
      |=~, matches|Pattern match|
      |!~, not_matches|Pattern mismatch|

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
- API Gateway Response Cache
  - 可以根据State分别设置Response Cache是否启用
  - Cache有效期间(TTL)范围: 0~300. 0代表Cache无效
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
- 几种API的Authorizer类型<br>
  **REST API**<br>
  <img src=https://routescroll.github.io/authorizer-rest.png width=70% /><br>
  **WebSocket API**<br>
  <img src=https://routescroll.github.io/authorizer-ws.png width=40% /><br>
  **HTTP API**<br>
  <img src=https://routescroll.github.io/authorizer-http.png width=70% /><br>

- 关于 `Integration`
  - **AWS_PROXY & HTTP_PROXY**
    - 貌似这种带有Proxy的会把Client发来的Request直接发送给后端HTTP Endpoint或Lambda, 会把后端的Response直接发送给Client
    - 选择Lambda作为Integration时Proxy只能为 **AWS_PROXY** 且不能更改
  - **AWS & HTTP**
    - 貌似不带Proxy的需要自己定义某种映射, 把Request/Response重新映射成自己想要的格式, 再发送给后端/Client, 这个过程会发生对Reqeust/Reponse内容的改写?
  - 如果后端是其他的HTTP Endpoint, 需要选择HTTPXXX系列的Integration类型
  
  **发送给后端Lambda的Request的例子**<br>
  ```json
  {
    "resource": "/my/path",
    "path": "/my/path",
    "httpMethod": "GET",
    "headers": {
      "header1": "value1",
      "header2": "value2"
    },
    "multiValueHeaders": {
      "header1": [
        "value1"
      ],
      "header2": [
        "value1",
        "value2"
      ]
    },
    "queryStringParameters": {
      "parameter1": "value1",
      "parameter2": "value"
    },
    "multiValueQueryStringParameters": {
      "parameter1": [
        "value1",
        "value2"
      ],
      "parameter2": [
        "value"
      ]
    },
    "requestContext": {
      "accountId": "123456789012",
      "apiId": "id",
      "authorizer": {
        "claims": null,
        "scopes": null
      },
      "domainName": "id.execute-api.us-east-1.amazonaws.com",
      "domainPrefix": "id",
      "extendedRequestId": "request-id",
      "httpMethod": "GET",
      "identity": {
        "accessKey": null,
        "accountId": null,
        "caller": null,
        "cognitoAuthenticationProvider": null,
        "cognitoAuthenticationType": null,
        "cognitoIdentityId": null,
        "cognitoIdentityPoolId": null,
        "principalOrgId": null,
        "sourceIp": "IP",
        "user": null,
        "userAgent": "user-agent",
        "userArn": null,
        "clientCert": {
          "clientCertPem": "CERT_CONTENT",
          "subjectDN": "www.example.com",
          "issuerDN": "Example issuer",
          "serialNumber": "a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1",
          "validity": {
            "notBefore": "May 28 12:30:02 2019 GMT",
            "notAfter": "Aug  5 09:36:04 2021 GMT"
          }
        }
      },
      "path": "/my/path",
      "protocol": "HTTP/1.1",
      "requestId": "id=",
      "requestTime": "04/Mar/2020:19:15:17 +0000",
      "requestTimeEpoch": 1583349317135,
      "resourceId": null,
      "resourcePath": "/my/path",
      "stage": "$default"
    },
    "pathParameters": null,
    "stageVariables": null,
    "body": "Hello from Lambda!",
    "isBase64Encoded": false
  }
  ```

## DynamoDB
- `Partition Key` & `Primary Key`
  - 官方定义
    - `Partition key`
      > The partition key is part of the table's primary key. It is a hash value that is used to retrieve items from your table and allocate data across hosts for scalability and availability.
    - `Sort key` - optional
      > You can use a sort key as the second part of a table's primary key. The sort key allows you to sort or search among all items sharing the same partition key.
  - `Primary Key` 用于唯一标识一个项目. 可以是 `Partition Key`, 也可以是 `Partition Key` + `SortKey`
  - `Partition Key` 用于决定Item保存在哪个Patition中(DynamoDB内部处理, 所有Items都会根据Partition Key的Hash值分散在各个Partition中保存)
    <img name=dynamo-partition src=https://routescroll.github.io/dynamo-partition.jpeg width=50%>
- DynamoDB Stream
  - `按修改时间顺序保存Item修改记录,Max 24小时以内记录`
  - Stream Invoke Lambda 是 synchronously
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
  - 使用 `Limit Parameter` 限制 **Request返回的最大结果数量**
  - `Parallel Scan` 的使用场景
    -  Table Size > `20GB`
    -  Table `Provisioned Read Throughput` 没有被充分利用
    -  顺序Scan过慢(Sequential Scan)
  -  > 默认情况下，Scan 操作按顺序处理数据。Amazon DynamoDB 以 1 MB 的增量向应用程序返回数据，应用程序执行其他 Scan 操作检索接下来 1 MB 的数据。扫描的表或索引越大，Scan 完成需要的时间越长。此外，一个顺序 Scan 可能并不总是能够充分利用预置读取吞吐量容量：即使 DynamoDB 跨多个物理分区分配大型表的数据，Scan 操作一次只能读取一个分区。出于这个原因，Scan 受到单个分区的最大吞吐量限制。为了解决这些问题，Scan操作可以逻辑地将表或二级索引分成多个分段，多个应用程序工作进程并行扫描这些段。每个工作进程可以是一个线程（在支持多线程的编程语言中），也可以是一个操作系统进程。要执行并行扫描，每个工作进程都会发出自己的 Scan 请求，并使用以下参数：
    Segment — 要由特定工作进程扫描的段。每个工作进程应使用不同的 Segment 值。
    TotalSegments — 并行扫描的片段总数。该值必须与应用程序将使用的工作进程数量相同。
    <img name=parallelscan src=https://routescroll.github.io/ParallelScan.png width=50% />
  - nodejs使用例<br>
    ```JavaScript
    Scan(TotalSegments=4, Segment=0, ...)
    Scan(TotalSegments=4, Segment=1, ...)
    Scan(TotalSegments=4, Segment=2, ...)
    Scan(TotalSegments=4, Segment=3, ...)
    ```
  - CLI使用例
    `aws dynamodb scan --totalsegments x --segment y ...`
  - Request Sample
    ```json
    {
      "AttributesToGet": [ "string" ],
      "ConditionalOperator": "string",
      "ConsistentRead": boolean,
      "ExclusiveStartKey": { 
          "string" : { 
            "B": blob,
            "BOOL": boolean,
            "BS": [ blob ],
            "L": [ 
                "AttributeValue"
            ],
            "M": { 
                "string" : "AttributeValue"
            },
            "N": "string",
            "NS": [ "string" ],
            "NULL": boolean,
            "S": "string",
            "SS": [ "string" ]
          }
      },
      "ExpressionAttributeNames": { 
          "string" : "string" 
      },
      "ExpressionAttributeValues": { 
          "string" : { 
            "B": blob,
            "BOOL": boolean,
            "BS": [ blob ],
            "L": [ 
                "AttributeValue"
            ],
            "M": { 
                "string" : "AttributeValue"
            },
            "N": "string",
            "NS": [ "string" ],
            "NULL": boolean,
            "S": "string",
            "SS": [ "string" ]
          }
      },
      "FilterExpression": "string",
      "IndexName": "string",
      "Limit": number, #---------------Maximum iteme number of request
      "ProjectionExpression": "string",
      "ReturnConsumedCapacity": "string",
      "ScanFilter": { 
          "string" : { 
            "AttributeValueList": [ 
                { 
                  "B": blob,
                  "BOOL": boolean,
                  "BS": [ blob ],
                  "L": [ 
                      "AttributeValue"
                  ],
                  "M": { 
                      "string" : "AttributeValue"
                  },
                  "N": "string",
                  "NS": [ "string" ],
                  "NULL": boolean,
                  "S": "string",
                  "SS": [ "string" ]
                }
            ],
            "ComparisonOperator": "string"
          }
      },
      "Segment": number, #---------------Parallel Scan Segment Number of an individual segment
      "Select": "string",
      "TableName": "string",
      "TotalSegments": number #---------------Parallel Scan Segment Total Number
    }
    ```
- `Scan操作对于RCU的冲击`
  <img name=scan src=https://routescroll.github.io/GetImage.jpeg width=50% /><br>
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
- DynamoDB不支持Resource-Based Policy
  - Identity-Based Polic允许用户访问某个或某些Talbe
- 通过 `IAM Policy` 可以限制用户只能访问DynamoDB Talbe的某些Item/某些Attributes
  - **考虑一款手机游戏应用，让玩家可以选择和玩各种不同的游戏。该应用程序使用一个名为GameScores的DynamoDB表来跟踪高分和其他用户数据。表中的每个项都由用户ID和用户所玩游戏的名称唯一标识。GameScores表有一个主键，由分区键(UserId)和排序键(GameTitle)组成。用户只能访问与其用户ID相关的游戏数据。想要玩游戏的用户必须属于名为GameRole的IAM角色，该角色有一个附加的安全策略。要在这个应用程序中管理用户权限，你可以写一个权限策略，如下所示:**
    ```yaml
    {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowAccessToOnlyItemsMatchingUserID",
              "Effect": "Allow",
              "Action": [
                  "dynamodb:GetItem",
                  "dynamodb:BatchGetItem",
                  "dynamodb:Query",
                  "dynamodb:PutItem",
                  "dynamodb:UpdateItem",
                  "dynamodb:DeleteItem",
                  "dynamodb:BatchWriteItem"
              ],
              "Resource": [
                  "arn:aws:dynamodb:us-west-2:123456789012:table/GameScores"
              ],
              "Condition": {
                  "ForAllValues:StringEquals": {
                      "dynamodb:LeadingKeys": [
                          "${www.amazon.com:user_id}"
                      ],
                      "dynamodb:Attributes": [
                          "UserId",
                          "GameTitle",
                          "Wins",
                          "Losses",
                          "TopScore",
                          "TopScoreDateTime"
                      ]
                  },
                  "StringEqualsIfExists": {
                      "dynamodb:Select": "SPECIFIC_ATTRIBUTES"
                  }
              }
          }
      ]
    }
    ```
  - **除了为GameScores表(Resource元素)上的特定DynamoDB动作(Action元素)授予权限外，Condition元素还使用以下特定于DynamoDB的条件键来限制权限，如下所示:**<br>
    - **dyanmodb:LeadingKeys** —这个条件键允许用户只访问分区键值与用户ID匹配的项目。这个ID ${www.amazon.com:user_id}是一个替换变量。有关替代变量的更多信息，请参见使用web身份联合。<br>
    - **dynamodb:Attributes** -这个条件键限制对指定属性的访问，这样只有权限策略中列出的操作才能返回这些属性的值。此外，StringEqualsIfExists子句确保应用程序必须始终提供一个特定属性的列表来进行操作，并且应用程序不能请求所有属性。评估IAM策略时，结果总是true(允许访问)或false(拒绝访问)。如果Condition元素的任何部分为假，则整个策略的计算结果为假，并拒绝访问。

  - **下面的权限策略允许对表和表索引(在Resource元素中指定)执行特定的DynamoDB操作(在Action元素中指定)。该策略使用dynamodb:LeadingKeys条件键将权限限制到分区键值与用户的Facebook ID匹配的项目**
  ```yaml
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "LimitAccessToCertainAttributesAndKeyValues",
            "Effect": "Allow",
            "Action": [
                "dynamodb:UpdateItem",
                "dynamodb:GetItem",
                "dynamodb:Query",
                "dynamodb:BatchGetItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-west-2:123456789012:table/GameScores",
                "arn:aws:dynamodb:us-west-2:123456789012:table/GameScores/index/TopScoreDateTimeIndex"
            ],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "dynamodb:LeadingKeys": [
                        "${graph.facebook.com:id}"
                    ],
                    "dynamodb:Attributes": [
                        "attribute-A",
                        "attribute-B"
                    ]
                },
                "StringEqualsIfExists": {
                    "dynamodb:Select": "SPECIFIC_ATTRIBUTES",
                    "dynamodb:ReturnValues": [
                        "NONE",
                        "UPDATED_OLD",
                        "UPDATED_NEW"
                    ]
                }
            }
        }
    ]
  }
  ```
  > **<font size=5 color=red>Important</font>**
  > 如果使用dynamodb:Attributes，则必须指定表的所有主键和索引键属性的名称(在**Resource**中指定)，以及策略中列出的任何辅助索引的名称。否则，DynamoDB不能使用这些关键属性来执行请求的操作。
- **GSI 与 RCU/WCU**
  - GSI 也需要分配RCU/WCU
  - PrimaryTable向IndexTable同步数据时会消耗Index的WCU
  - Index的WCU分配不足时会使PrimaryTable出现限流错误(Throttling Error)
  - 起码要为IndexTable分配和PrimaryTable一样多的WCU/RCU

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
- `/tmp`
  - 最大**512MB**
  - 同一Lambda每次Invocation时 `/tmp` 下的东西不变
  - 可以用作每次Invocation都要用但长期保持不变的文件的缓存
- Lambda连接VPC Private Subnet所需必要信息
  - Private Subnet ID
  - SG ID
    > Lambda连接VPC Private Subnet后为了能继续访问Internet, 需要给目标VPC添加NAT
- `alias`
  - 类似于Lambda的指针, 指向一个指定的Lambda Version
  - 也可以使 `alias` 指向两个不同的Version, 并分别指定两个Version的Traffic Weighting
    <img name=lambda-aliss-weighting src=https://routescroll.github.io/lambda-aliss-weighting.jpeg width=50%>
- CloudFormation template
  - 代码文件除了打包上传S3, 还可以在template.yaml中直接写代码(比如用Nodejs或者Python时)
    <img name=lambda-template-sourcecode src=https://routescroll.github.io/lambda-template-sourcecode.jpeg width=50%>
  - 使用S3保存代码时的写法
    <img name=lambda-template-s3 src=https://routescroll.github.io/lambda-template-s3.jpeg width=50%>
- Environment Variable的 `Transit Encryption`
  - 为Lambda添加 Environment Variable 时可以选择是否用 `Encrypt Helper` 对Environment Variable的Value进行加密, 并使用KMS作为加密/解密Key
  - 这样 Environment Variable 的Value在Console中会显示为加密后的字符串, Lambda代码中要使用该Environment Variable时需要通过KMS解密该 Environment Variable 的Value后才能使用<br>
    <img name=lambda-add-encrypt-env-val src=https://routescroll.github.io/lambda-add-encrypt-env-val.png width=50%><img name=lambda-encrypt-env-kms-samplecode src=https://routescroll.github.io/lambda-encrypt-env-kms-samplecode.png width=50%><img name=lambda-after-encrypt src=https://routescroll.github.io/lambda-after-encrypt.png width=50%>
- **Lambda Desitination**
  - For each execution status (i.e. **Success and Failure**), you can choose one destination from four options: 
    1. another Lambda function
    2. an SNS topic
    3. an SQS standard queue
    4. or EventBridge. 
- **Lambda不能创建Endpoint, 只能连接到VPC(或者说 Add a Lambda to the VPC)**

## CloudWatch
- Alarm
  - `High-Resolution Alarm`
    - 频度:10s / 30s / 60s整数倍
  - `High-Resolutin Metric`
    - 1/5/10/30/60s/60s的整数倍
  - `Detailed Monitoring`
    - 频度只有1min, 没有更细的频度
- Metric Filters
  - 用户自行创建 `Metric Filters` 可以在CloudWatch Log中搜索期望的值并数字化后作为Metric中的一维显示或用其设置Alarm
  - <font color=red>需要注意</font>: CloudWatch只会获取 `Metric Filters` 创建以后发生的Metric数据, 在此之前的Log不是新 `Metric Filter` 的查找对象
- CloudWatch Event
  - 使用`CloudWatch Event Rule`获取Service的各种变化的数据(metadata), 并传递给Targe(eg. Lambda)<br>
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
- `namespace`
  - 不同 `namespace` 中的Metrics是彼此独立的
  - 故, 给各个application定义自己的 `namespace`, 把各个application各自的Metrics发送到各自的 `namespace` 中, 就不会把很多application的Metrics混在一起显示了

## AppSync - GraphQL
- GraphQL与AppSync组合向用户提供 **Single API - Multi Data Source** 服务<br>
![appsync](https://routescroll.github.io/graphql.jpeg)

## Elastic Beanstalk
- 全托管
  - `Capacity Provisioning`
  - `Load Balancing`
  - `Auto Scaling`
  - `RDS Integrated (RDS Auto Scaling included)`
- 关于配制
  - 对于Configuration File的要求

    |Request Item|RequestDetails|
    |-|-|
    |Location|Folder Name:**`.ebextensions`**<br>Folder Location: root of Source Bundle<br>Files Location: **`.ebextensions`**<br>|
    |Naming|Configuration files must have the **`.config`** file extension|
    |Formatting|Configuration files must conform to **`YAML or JSON`** specifications|
    |Uniqueness|Use **`each key only once`** in **`each configuration file`**|

    ![ebs-config-files](https://routescroll.github.io/ebs-config-files.jpeg)
  - **dockerrun.aws.json**
    - 描述如何将远程Docker Image部署为一个Elastic Beanstalk Application
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
  - Deploy比较
    **Deploy失败的时候有一部分是需要手动Redeploy的**
    ![elastic-beanstalk-deploy](https://routescroll.github.io/elastic-beanstalk-deploy.jpeg)
  - Deploy图示 ([参考](https://dev.classmethod.jp/articles/elastic-beanstalk-deploy-policy/))
    - **<font color=red>All-at-once</font>**<br>
    <img src=https://routescroll.github.io/all_at_once1.png width=50%>
    <img src=https://routescroll.github.io/all_at_once2.png width=50%>
    <img src=https://routescroll.github.io/all_at_once3.png width=50%>
    - **<font color=red>Rolling</font>**<br>
    <img src=https://routescroll.github.io/rolling1.png width=50%>
    <img src=https://routescroll.github.io/rolling2-1.png width=50%>
    <img src=https://routescroll.github.io/rolling3.png width=50%>
    <img src=https://routescroll.github.io/rolling4-2.png width=50%>
    <img src=https://routescroll.github.io/rolling5.png width=50%>
    - **<font color=red>Rolling additional Batch</font>**<br>
    <img src=https://routescroll.github.io/rolling_add1.png width=50%>
    <img src=https://routescroll.github.io/rolling_add2.png width=50%>
    <img src=https://routescroll.github.io/rolling_add6.png width=50%>
    <img src=https://routescroll.github.io/rolling_add3.png width=50%>
    <img src=https://routescroll.github.io/rolling_add4.png width=50%>
    <img src=https://routescroll.github.io/rolling_add5.png width=50%>
    - **<font color=red>Immutable</font>**<br>
    <img src=https://routescroll.github.io/immutable1.png width=50%>
    <img src=https://routescroll.github.io/immutable2-640x294.png width=50%>
    <img src=https://routescroll.github.io/immutaable3-640x294.png width=50%>
    <img src=https://routescroll.github.io/immutable4-640x294.png width=50%>
    <img src=https://routescroll.github.io/immutable5-640x294.png width=50%>
    <img src=https://routescroll.github.io/immutable6.png width=50%>

- 关于定义Container
  - 必要文件: <font color=red>Dockerrun.aws.json</font>
  - 文件路径: Source Bundle的root路径下. (在EBS Instance里位于 <font color=red>/var/app/current/</font>)
  - 必要section:
    1. AWSEBDockerrunVersion
      > Specifies the version number as the value 2 for ECS managed Docker environments.
    2. containerDefinitions
      > An array of container definitions, detailed below.
    3. volumes
      > Creates volumes from folders in the Amazon EC2 container instance, or from your source bundle (deployed to /var/app/current).<br>
      > <font color=red>Mount these volumes to paths within your Docker containers using **mountPoints**</font>
    ```json
    {
      "AWSEBDockerrunVersion": 2,
      "volumes": [
        {
          "name": "php-app",
          "host": {
            "sourcePath": "/var/app/current/php-app"
          }
        },
        {
          "name": "nginx-proxy-conf",
          "host": {
            "sourcePath": "/var/app/current/proxy/conf.d"
          }
        }
      ],
      "containerDefinitions": [
        {
          "name": "nginx-proxy",
          "image": "nginx",
          "essential": true,
          "memory": 128,
          "portMappings": [
            {
              "hostPort": 80,
              "containerPort": 80
            }
          ],
          "links": [
            "php-app"
          ],
          "mountPoints": [
            {
              "sourceVolume": "php-app",
              "containerPath": "/var/www/html",
              "readOnly": true
            },
            {
              "sourceVolume": "nginx-proxy-conf",
              "containerPath": "/etc/nginx/conf.d",
              "readOnly": true
            },
            {
              "sourceVolume": "awseb-logs-nginx-proxy",
              "containerPath": "/var/log/nginx"
            }
          ]
        }
      ]
    }
    ```

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
- **Annotation** & **Metadata**
  - **Annotation**
    - 用来记录想要用来分组的traces, 可以Index, 可以Search, 可以Filter
  - **Metadata**
    - 用来记录用户希望保存到traces的追加数据, 不可Index, 不可Search, 不可Filter
  - 追加目标
    - 由 **X-Ray** 创建的 **segment**
    - 由 **用户自定义** 的 **subsegment**
  

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
- 关于 `SCPs`
  - `SCPs` 用来控制用户权限(Service Control Policies)
    ![scps](https://routescroll.github.io/scps.png)
  - 通过SCPs可以集中控制Organization中所有Account的最大权限
    - 两种控制方法
      1. **Deny List** 各种Action默认为Allow, 需要指定 **What Services & What Actions** 被Deny
      ```json
      {
        "Version": "2012-10-17",
        "Statement": [
          {
              "Sid": "AllowsAllActions",
              "Effect": "Allow",
              "Action": "*",
              "Resource": "*"
          },
          {
              "Sid": "DenyDynamoDB", 
              "Effect": "Deny",
              "Action": "dynamodb:*",
              "Resource": "*"
          }
        ]
      }
      ```
      2. **Allow List** 各种Action默认为Deny, 需要指定 **What Services & What Actions** 被Allow
      ```json
      {
        "Version": "2012-10-17",
        "Statement": [
          {
              "Sid": "DenyAllActions",
              "Effect": "Deny",
              "Action": "*",
              "Resource": "*"
          },
          {
              "Sid": "AllowDynamoDB", 
              "Effect": "Allow",
              "Action": "dynamodb:*",
              "Resource": "*"
          }
        ]
      }
      ```
- **IAM Policy**
  - IAM Policy 隐式 Deny All Access, 除非显式指定 Allow Some Access
- **Best Practic**
  - Use groups to assign permissions to users
  - Create standalone policies instead of using inline policies -> `Inline Policies` 只能应用于单一Entity, 不能被其他Entity复用

## AWS Cognito
- `User Pools` & `Identity Pools`

  |Name|Description|
  |-|-|
  |`User Pools`|用户群体用于身份验证（身份核实）。如果使用用户群体，应用程序用户将可以通过该用户群体进行登录或通过第三方身份提供者（IdP）进行联合身份验证。|
  |`Identity Pools`|身份池用于授权（访问控制）。您可以使用身份池为用户创建唯一身份并向他们授予访问其他 AWS 服务的权限。|

- `Cognito` 所支持的`Identity Providers`
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
- `Cognito User Pool提供登录页面`
  > When you create a user pool in Amazon Cognito and then configure a domain for it, Amazon Cognito automatically provisions a hosted web UI to let you add sign-up and sign-in pages to your app. You can add a custom logo or customize the CSS for the hosted web UI.
  1. 在Cognito中创建 `user pool`
  2. 为该 `user pool` 配置一个domain
  3. Cognito会为你的app自动提供一个Web UI供用户注册登录. 用户可以向该页面添加自己的CSS或者logo
- `Cognito Sync`
  - 用户Deviece中使用 `Client Library` 可以Device内部建立 **Local Cache of Identity Data**
  - `Client Library` 与云端Cognito Service进行数据同步, 从而使用户在多个设备上都能保持同样的Identity Data同步

- 关于 `Unauthenticated identities(users)`
  - Cognito **identity pools** 支持允许用户在不登录的情况下获取有限的访问权限, 设置简单, 只需增加一个有限权限的Role并分配给 `Unauthenticated users`

## Step Function
- `Fields Filter` , 用于从InputJSON中选择特定项目使用
  - `InputPath` -> 用于选择输入JSON特定项目
  - `OutputPath` -> 用于选择传往下个状态JSON特定项目(从Input中选择)
  - `ResultPath` -> 用于选择最终输出JSON特定项目(从Input中选择)
    - 例如,原始Input JSON<br>
      ```json
      {
        "comment": "An input comment.",
          "data": {
          "val1": 23,
          "val2": 17
        },
        "extra": "foo",
        "lambda": {
          "who": "AWS Step Functions"
        }
      }
      ```
    - Step Function定义<br>
      ```json
      {
        "Comment": "A Catch example of the Amazon States Language using an AWS Lambda function",
        "StartAt": "CreateAccount",
        "States": {
          "CreateAccount": {
            "Type": "Task",
            "Resource": "arn:aws:lambda:us-east-1:123456789012:function:FailFunction",
            "InputPath": "\$.lambda", #----------真正输入到Function中的只有lambda项目
            "OutputPath": "\$.data", #----------传递个下一个状态Function的只有data项目
            "ResultPath": "\$.data.lambdaresult", #----------输出结果会放到data.lambdaresult下面,输出的内容只有data项目
            "Catch": [ {
              "ErrorEquals": ["CustomError"],
              "Next": "CustomErrorFallback",
              "ResultPath": "$.data.error", #----------发生异常时结果会放到data.error下面
            }, {
              "ErrorEquals": ["States.TaskFailed"],
              "Next": "ReservedTypeFallback"
            }, {
              "ErrorEquals": ["States.ALL"],
              "Next": "CatchAllFallback"
            } ],
            "End": true
          },
          "CustomErrorFallback": {
            "Type": "Pass",
            "Result": "This is a fallback from a custom Lambda function exception",
            "End": true
          },
          "ReservedTypeFallback": {
            "Type": "Pass",
            "Result": "This is a fallback from a reserved error code",
            "End": true
          },
          "CatchAllFallback": {
            "Type": "Pass",
            "Result": "This is a fallback from any error code",
            "End": true
          }
        }
      }
      ```
    - 回顾 - 原始输入JSON<br>
      ```json
      {
        "comment": "An input comment.",
        "data": {
          "val1": 23,
          "val2": 17
        },
        "extra": "foo",
        "lambda": {
          "who": "AWS Step Functions"
        }
      }
      ```
    - 过滤后Input:<br>
      ```json 
      {
        {
          "who": "AWS Step Functions"
          }
      }
      ```
    - 过滤后Output:<br>
      ```json
      {
        "val1": 23,
        "val2": 17
      }
      ```
    - 过滤后Result:<br>
      ```json
      {
        "val1": 23,
        "val2": 17,
        "lambdaresult": "some lambda results"
      }
      ```
    - 过滤后Exception:<br>
      ```json
      {
        "val1": 23,
        "val2": 17,
        "error": "error message"
      }
      ```

- `Parameters` -> 从Input中选择某些项目重新组合成新的Input内容<br>
  - 原始json<br>
    ```json
    {
      "comment": "Example for Parameters.",
      "product": {
        "details": {
        "color": "blue",
        "size": "small", #--------映射对象:size
        "material": "cotton"
        },
        "availability": "in stock", #--------映射对象:availability
        "sku": "2317",
        "cost": "$23"
      }
    }
    ```
  - 使用Parameters重新映射成新Input<br>
    ```json 
      "Parameters": {
      "comment": "Selecting what I care about.",
      "MyDetails": {
        "size.\$": "\$.product.details.size", #--------size -> size
        "exists.\$": "\$.product.availability", #-------- availability -> exists
        "StaticValue": "foo"
      }
    }
    ```
  - 映射后的新Input<br>
    ```json
    {
      "comment": "Selecting what I care about.",
      "MyDetails": {
        "size": "small", ###
        "exists": "in stock", ###
        "StaticValue": "foo"
      }
    }
    ```
- `ResultSelector` 从输出结果中选择特定项目,应用于`ResultPath`<br>
  - 原始Result<br>
    ```json
    {
      "resourceType": "elasticmapreduce", #--------被选择项目
      "resource": "createCluster.sync",
      "output": {
        "SdkHttpMetadata": {
            "HttpHeaders": {
              "Content-Length": "1112",
              "Content-Type": "application/x-amz-JSON-1.1",
              "Date": "Mon, 25 Nov 2019 19:41:29 GMT",
              "x-amzn-RequestId": "1234-5678-9012"
            },
            "HttpStatusCode": 200
          },
          "SdkResponseMetadata": {
            "RequestId": "1234-5678-9012"
        },
        "ClusterId": "AKIAIOSFODNN7EXAMPLE" #--------被选择项目
      }
    }
  - 应用`ResultSelector`<br>
    ```json
    "Create Cluster": {
      "Type": "Task",
      "Resource": "arn:aws:states:::elasticmapreduce:createCluster.sync",
      "Parameters": {
        "snowsome parameters"
      },
      "ResultSelector": {
        "ClusterId.$": "$.output.ClusterId", #--------被选择项目放在了这里
        "ResourceType.$": "$.resourceType" #--------被选择项目放在了这里
      },
      "ResultPath": "$.EMROutput",
      "Next": "Next Step"
    }
    ```
  - 最终 Result<br>
    ```json 
    {
      "OtherDataFromInput": {},
      "EMROutput": {
        "ResourceType": "elasticmapreduce", ###
        "ClusterId": "AKIAIOSFODNN7EXAMPLE" ###
      }
    }
    ```

- 可能的 `Destination`
    1. `SNS`
    2. `SQS`
    3. `Lambda`
    4. `EventBridge`

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
- `Systems Manager Parameter Store` 不会自动轮换加密证书之类的,需要application自己去做轮换
- [参考:Difference between Parameter Store & Secrets Manager](https://medium.com/awesome-cloud/aws-difference-between-secrets-manager-and-parameter-store-systems-manager-f02686604eae)
  > **Secrets Manager**: It was designed specifically for **confidential information (like database credentials, API keys)** that needs to be encrypted, so the creation of a secret entry has encryption enabled by default. It also gives **additional functionality** like **rotation of keys**.<br>
  > **Systems Manager** Parameter Store: It was designed to cater to a **wider use case**, not just secrets or passwords, but also application configuration variables like URLs, Custom settings, AMI IDs, License keys, etc.

  |Service|Cost|Rotation|Cross-account Access|Secret Size|Limits|Multiple Regions Replication|Use Cases|
  |-|-|-|-|-|-|-|-|
  |Secrets Manager|Paid|- Integration with<br>RDS<br>Redshift<br>DocumentDB<br>- By Lambda|Yes|Max 10KB|500000 secrets/region & account|Yes|store **only encrypted** values and super **easy** way to manage the **rotation** of the secrets|-|-|
  |Parameter Store|Paid when<br>- Higher Throughput<br>- Advanced Parameters|Only by Lambda|No|- Standard: 4KB<br>- Advanced: 8KB|10000 standard parameters/region & account|No|**cheaper** option to store **encrypted** or **unencrypted** secrets|-|-|

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
  - KMS的CMK只能直接加密不超过4096字节的数据, 所以不适于用来加密大型数据, 只能用于加密Key这类的小数据
  - 要加密大型数据还是要使用Data Key<br><br>
    用户指定一个对称加密KMS Key, 用于加密生成的`Data Key`
    ![generate-data-key](https://routescroll.github.io/generate-data-key.jpeg)

    使用Plaintext Data Key加密数据
    ![generate-data-key](https://routescroll.github.io/encrypt-data.jpeg)
- 过于频繁的的请求 `KMS GenerateDataKey` 可能会引发KMS API限流
  - **解决方法1**: 向AWS申请提高API请求限额
  - **解决方法2**: 使用 `AWS Encryption SDK` 并开启其 `LocalCryptoMaterialsCache`
    - 应用场景
      > - It can reuse data keys (**允许重复使用Data Keys加密**)
      > - It generates numerous data keys (**需要生成大量Data Keys**)
      > - Your cryptographic operations are unacceptably slow, expensive, limited, or resource-intensive (**加密操作需要快速/低价/有限的?/资源敏感**)
    - 使用方法
      > - **Java/Python** <u>LocalCryptoMaterialsCache constructor</u>
      > - **JavaScript** <u>getLocalCryptographicMaterialsCache function</u>
      > - **C** <u>aws_cryptosdk_materials_cache_local_new constructor</u>

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
    ```json
    {
      "Version": "2012-10-17",
      "Id": "PutObjectPolicy",
      "Statement": [{
        "Sid": "DenyIncorrectEncryptionHeader",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:PutObject",
        "Resource": "arn:aws:s3:::awsexamplebucket1/*",
        "Condition": {
          "StringNotEquals": {
            "s3:x-amz-server-side-encryption": "AES256"
          }
        }
      }]
    }
    ```
    > <br>
  - **S3 Security Transport**
    - 使用 Bucket Policy 强制访问S3使用加密<br>
    <img name=s3-security-transit src=https://routescroll.github.io/s3-security-transit.jpeg width=50%>
- **S3 Transfer Acceleration**
  - 借用CloudFront加快用户上传文件到S3的速度
- **S3 Policy Variable**
  - **${xxxxx}** 就是这个所谓的Variable, 可以被花括号内的内容动态替换<br>
  <img name=s3-policy-variable src=https://routescroll.github.io/s3-policy-variable.jpeg width=50%>


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
- **Change Set**
  - 创建Change Set, 并且可以 **View Change Set** 从而在真正应用变更之前可以明确更改会对当前Resource造成哪些影响, 进而决定是否应用本次更改或做进一步变更后再应用更改
- **Stack Cross-Account**
  - Stack可以 Cross-Account 进行部署<br>
    <img name=cloudformation-stack-cross-account src=https://routescroll.github.io/cloudformation-stack-cross-account.jpeg width=50%>


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
- 使用 `launch configuration`
  - `launch configuration` **可以指定ASG**
  - 想更改 `launch configuration`, 只能建立新版本, 不能直接改已有的
- **Default Credential Provider Chain** (不仅仅是EC2, 应该说时AWS SDK/CLI 的共通设置) 
  (**When you initialize a new service client without supplying any arguments**)
  Java AWS SDK/CLI 的 `Credential Provider Chain` 查找顺序
    1. **Environment variables** – AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY (Java SDK API: `EnvironmentVariableCredentialsProvider`)
    2. **Java system properties** – aws.accessKeyId and aws.secretKey (Java SDK API: `SystemPropertiesCredentialsProvider`)
    3. **The default credential profiles file** -  typically located at ~/.aws/credentials (location can vary per platform) (`ProfileCredentialsProvider`)
    4. **Amazon ECS container credentials** – loaded from the Amazon ECS if the environment variable AWS_CONTAINER_CREDENTIALS_RELATIVE_URI is set (`ContainerCredentialsProvider`)
    5. **Instance profile credentials** – used on EC2 instances and delivered through the Amazon EC2 metadata service. (`InstanceProfileCredentialsProvider`)

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
- `File Invalidation`
  - 当用户发现不能刷新到新的文件时(比如图片), 需要Service端(`CloudFront`)进行旧版本文件的无效化
  - 该操作需要在 `CloudFront` 端进行, 指定那些文件需要无效化

## VPC
- `VPC Flow Logs`
  - 应用场景
    > Diagnosing overly restrictive security group rules (**诊断过于严格的SG规则**)<br>
    > Monitoring the traffic that is reaching your instance (**监控到达Instance的流量**)<br>
    > Determining the direction of the traffic to and from the network interfaces (**确定进出网络接口的流量方向**)<br>
  - 三种监控级别: <u>Instance</u>, <u>Subnet</u>, <u>VPC</u>
    ![vpc-flow-level](https://routescroll.github.io/vpc-flow-level.png)
  
## Kinesis
- Structure<br>
  <img name=kniesis-structure src=https://routescroll.github.io/kniesis-structure.jpeg width=50%>
- `Kinesis Client Library (KCL)` 运行在Consumer EC2集群里, 负责处理数据
  ![kinesis-flow.png](https://routescroll.github.io/kinesis-flow.png)
- `Kinesis Producer Library (KPL)` 运行在Producer端, 可以协助Producer端更好的利用Shards Throughput
  1. 自动向1个或多个Data Stream写入数据并且带有可配制的Retry机制
  2. 可以使用 `PutRecords` 在一次Request中向多个Shards写入多个Records
  3. 与KCL配合在Consumer端分解Batched Data
  4. 向CloudWatch写入Metric以提供Producer性能的可视化
- 支持 Server-side 加密, 从Stream写入Storage Layer之前会被加密, 从Stroage Layer取出的时候会被解密
  - 支持使用KMS加密
- `Shard` 中的数据默认保持24小时, 最高保持168小时后就会从Shard中删除
- `Shards` 与 `Worker/Processor` 的数量关系
  `Worker/Processor` 的数量不能多于(没必要多于) Shards数量, 一个一个Shard对应一个EC2 `Worker/Processor`, 但是一个 `Worker/Processor` 可以对应多个Shards<br>
  <img name=kinesis-shards-workers src=https://routescroll.github.io/kinesis-shards-workers.png width=50%>
- Stream重新分片
  - Scale Up/Down 的目标Shard值推荐为当前Shard值的 25%/50%/75%/100%. 指定其他比例的值可能会导致重新分片更慢
  - 一些限制
    1. **每24小时重**新分片次数不**超过10次**/Stream
    2. Scale up 不超过当前Shard数的**2倍**/Stream
    3. Scale down 不超过当前Shard数的一半/Stream
    4. Scale up 不超过500 Shards/Stream
    5. Scale down 不超过500, 除非 Scale down后的数量低于500
    6. Scale up 不超过当前账户Shard数限制
- **Fireose**<br>
  <img name=kenesis-firehose src=https://routescroll.github.io/kenesis-firehose.png width=50%>

  - 支持的存储目标
    1. S3
    2. RedShift
    3. ElasticSearch
    4. splunk

## STS (Security Token Service)
- Typically, you use **`GetSessionToken`** if you want to **`use MFA`** to **`protect programmatic calls to`** specific **`AWS API`** operations
- 使用 **STS的`decode-authorization-message`** API 来解码授权相关的加密Message

## CloudTrail
- 貌似通过Console创建的CloudTrail无需每个Region都设置一次CloudTrail. 默认都是应用于 `All Region`, 记录下来的log会同一存储在同一个S3 Bucket里

## ElasticCache
- **Memcached 和 Redis 的对比**<br>
  <img name=elastic-cache-compare src=https://routescroll.github.io/elastic-cache-compare.png width=50%>

## SQS
- 关于Message `VisibilityTimeout` 可见期间
  - 如果Messenger的Consumer在 `VisibilityTimout` 期间没有完成处理, 那么Message会再次出现在Queue中, 有可能会被其他Consumer接收到, 造成同一Message重复处理<br>
  <img name=sqs-visibility-timeout src=https://routescroll.github.io/sqs-visibility-timeout.png width=50%>
- Amazon SQS only supports messages up to **256KB** in size
  - Message高于256KB情况下需要用S3存储Message, 并且配合 **SQS Extended Client Library for Java** 来管理Message

## AWS Batch
- 需要EC2支持, 不属于Serverless

## AWS Shield DDoS防火墙
## AWS Inspector
  - 对于部署在AWS上的Application进行安全评估(漏洞, 脆弱性, 偏差)
  - 生成安全问题列表(以安全等级排序)
  - 被发现的安全问题可以直接Review, 也可以作为评估报告详细的一部分(通过Inspector Console or API)
---

# 个别题型
### Test3-Incorrect-问题 56： 不正确 

A developer is creating an Auto Scaling group of Amazon EC2 instances. The developer needs to publish a custom metric to Amazon CloudWatch. Which method would be the MOST secure way to authenticate a CloudWatch PUT request? 

Create an IAM role with the PutMetricData permission and create a new Auto Scaling launch configuration to launch instances using that role 

（正确） 

Create an IAM role with the PutMetricData permission and modify the Amazon EC2 instances to use that role 

（错误） 

Modify the CloudWatch metric policies to allow the PutMetricData permission to instances from the Auto Scaling group 

Create an IAM user with the PutMetricData permission and modify the Auto Scaling launch configuration to inject the user credentials into the instance user data 

注解 

The most secure configuration to authenticate the request is to create an IAM role with a permissions policy that only provides the minimum permissions requires (least privilege). This IAM role should have a customer-managed permissions policy applied with the PutMetricData allowed. 

The PutMetricData API publishes metric data points to Amazon CloudWatch. CloudWatch associates the data points with the specified metric. If the specified metric does not exist, CloudWatch creates the metric. When CloudWatch creates a metric, it can take up to fifteen minutes for the metric to appear in calls to ListMetrics. 

The following images shows a permissions policy being created with the permission: 
...

CORRECT: "Create an IAM role with the PutMetricData permission and create a new Auto Scaling launch configuration to launch instances using that role" is the correct answer 

INCORRECT: "Modify the CloudWatch metric policies to allow the PutMetricData permission to instances from the Auto Scaling group" is incorrect as this is not possible. You should instead grant the permissions through a permissions policy and attach that to a role that the EC2 instances can assume. 

INCORRECT: "Create an IAM user with the PutMetricData permission and modify the Auto Scaling launch configuration to inject the user credentials into the instance user data" is incorrect. You cannot “inject user credentials” using a launch configuration. Instead, you can attach an IAM role which allows the instance to assume the role and take on the privileges allowed through any permissions policies that are associated with that role. 

INCORRECT: "Create an IAM role with the PutMetricData permission and modify the Amazon EC2 instances to use that role" is incorrect as you should <font color=red>create a new launch configuration for the Auto Scaling group rather than updating the instances manually.</font>

References: 

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html 

https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/aws-iam/ 

https://digitalcloud.training/amazon-ec2/ 

### Test3-Skipped-问题 23：要回看教程 DynamoDB

跳过 

A gaming company is building an application to track the scores for their games using an Amazon DynamoDB table. Each item in the table is identified by a partition key 
(user_id) and a sort key (game_name). The table also includes the attribute “TopScore”. The table design is shown below: 

<img src=https://routescroll.github.io/test3-23.jpeg width=50%>

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

<img src=https://routescroll.github.io/test3-23-index.jpeg width=50%>

CORRECT: "Create a global secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute" is the correct answer. 

INCORRECT: "Create a local secondary index with a partition key of “game_name” and a sort key of “TopScore” and get the results based on the score attribute" is incorrect. With a local secondary index you can have a different sort key but the partition key is the same. 

INCORRECT: "Use a DynamoDB scan operation to retrieve the scores for “game_name” using the “TopScore” attribute, and order the results based on the score attribute" is incorrect. This would be inefficient as it scans the whole table. First, we should create a global secondary index, and then use a query to efficiently retrieve the data. 

INCORRECT: "Create a global secondary index with a partition key of “user_id” and a sort key of “game_name” and get the results based on the score attribute" is incorrect as with a global secondary index you have a different partition key and sort key. Also, we don’t need “user_id”, we need “game_name” and “TopScore”. 

References: 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/amazon-dynamodb/ 

### Test3-Skipped-问题 54：要回看教程 DynamoDB

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


### Test5-Skipped-问题 22：  要回看教程 DynamoDB

跳过 

A Developer needs to return a list of items in a global secondary index from an Amazon DynamoDB table. 

Which DynamoDB API call can the Developer use in order to consume the LEAST number of read capacity units? 

Query operation using eventually-consistent reads 

（正确） 

Query operation using strongly-consistent reads 

Scan operation using strongly-consistent reads 

Scan operation using eventually-consistent reads 

注解 

The Query operation finds items based on primary key values. You can query any table or secondary index that has a composite primary key (a partition key and a sort key). 

For items up to 4 KB in size, one RCU equals one strongly consistent read request per second or two eventually consistent read requests per second. Therefore, using eventually consistent reads uses fewer RCUs. 

CORRECT: "Query operation using eventually-consistent reads" is the correct answer. 

INCORRECT: "Query operation using strongly-consistent reads" is incorrect as strongly-consistent reads use more RCUs than eventually consistent reads. 

INCORRECT: "Scan operation using eventually-consistent reads" is incorrect. The Scan operation returns one or more items and item attributes by accessing every item in a table or a secondary index and therefore uses more RCUs than a query operation. 

INCORRECT: "Scan operation using strongly-consistent reads" is incorrect. The Scan operation returns one or more items and item attributes by accessing every item in a table or a secondary index and therefore uses more RCUs than a query operation. 

References: 

https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html 

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/amazon-dynamodb/ 

### Test5-Skipped-问题 24：  要回看教程 X-Rays

跳过 

An application is being instrumented to send trace data using AWS X-Ray. A Developer needs to upload segment documents using JSON-formatted strings to X-Ray using the API. Which API action should the developer use? 

The PutTelemetryRecords API action 

The GetTraceSummaries API action 

The PutTraceSegments API action 

（正确） 

The UpdateGroup API action 

注解 

You can send trace data to X-Ray in the form of segment documents. A segment document is a JSON formatted string that contains information about the work that your application does in service of a request. Your application can record data about the work that it does itself in segments, or work that uses downstream services and resources in subsegments. 

Segments record information about the work that your application does. A segment, at a minimum, records the time spent on a task, a name, and two IDs. The trace ID tracks the request as it travels between services. The segment ID tracks the work done for the request by a single service. 

Example Minimal complete segment: 

"name" : "Scorekeep" , 
"id" 
: "7Øde5b6f19ff9aOa" , 
" start _ time" 
1.4782933612710, 
"trace_id" 
: "1-581cf771-aØ06649127e371903a2de979" , 
"end_time" 
1 .4782933614490 
You can upload segment documents with the PutTraceSegments API. The API has a single parameter, TraceSegmentDocuments, that takes a list of JSON segment documents. 

Therefore, the Developer should use the PutTraceSegments API action. 

 
 

CORRECT: "The PutTraceSegments API action" is the correct answer. 

INCORRECT: "The PutTelemetryRecords API action" is incorrect as this is used by the AWS X-Ray daemon to upload telemetry. 

INCORRECT: "The UpdateGroup API action" is incorrect as this updates a group resource. 

INCORRECT: "The GetTraceSummaries API action" is incorrect as this retrieves IDs and annotations for traces available for a specified time frame using an optional filter. 

References: 

https://docs.aws.amazon.com/xray/latest/devguide/xray-api-sendingdata.html 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/aws-developer-tools/ 

### Test3-Skipped-问题 49：要回看教程 X-Rays

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

### Test3-Skipped-问题 53：要回看教程 X-Rays

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

### Test6-Skipped-问题 36：要回看教程 X-Rays

跳过 

A Development team wants to instrument their code to provide more detailed information to AWS X-Ray than simple outgoing and incoming requests. This will generate large amounts of data, so the Development team wants to implement indexing so they can filter the data. 

What should the Development team do to achieve this? 

Install required plugins for the appropriate AWS SDK 

Configure the necessary X-Ray environment variables 

Add annotations to the segment document 

（正确） 

Add metadata to the segment document 

注解 

AWS X-Ray makes it easy for developers to analyze the behavior of their production, distributed applications with end-to-end tracing capabilities. You can use X-Ray to identify performance bottlenecks, edge case errors, and other hard to detect issues. 

When you instrument your application, the X-Ray SDK records information about incoming and outgoing requests, the AWS resources used, and the application itself. You can add other information to the segment document as annotations and metadata. Annotations and metadata are aggregated at the trace level and can be added to any segment or subsegment. 

Annotations are simple key-value pairs that are indexed for use with filter expressions. Use annotations to record data that you want to use to group traces in the console, or when calling the GetTraceSummaries API. X-Ray indexes up to 50 annotations per trace. 

Metadata are key-value pairs with values of any type, including objects and lists, but that are not indexed. Use metadata to record data you want to store in the trace but don't need to use for searching traces. 

You can view annotations and metadata in the segment or subsegment details in the X-Ray console. 

Subsegment - ## GameModel.saveGame 
Overview 
-resources- 
-gane-: 
Resources 
Annotations 
Metadata 
Exceptions 
-session-: "IC6r.m1DN% 
-nane-: gane% 
-users- 
-mEKIOt9L% 
"S7Q90REO- 
-rules-: "TicTacToe-, 
-start_tine-: 148953665359, 
-states- : 
"'SS37H81% 
-m87PSQEH" 
x 
Close 
In this scenario, we need to add annotations to the segment document so that the data that needs to be filtered is indexed. 

CORRECT: "Add annotations to the segment document" is the correct answer. 

INCORRECT: "Add metadata to the segment document" is incorrect as metadata is not indexed for filtering. 

INCORRECT: "Configure the necessary X-Ray environment variables" is incorrect as this will not result in indexing of the required data. 

INCORRECT: "Install required plugins for the appropriate AWS SDK" is incorrect as there are no plugin requirements for the AWS SDK to support this solution as the annotations feature is available in AWS X-Ray. 

References: 

https://docs.aws.amazon.com/xray/latest/devguide/xray-concepts.html#xray-concepts-annotations 

Save time with our AWS cheat sheets: 

https://digitalcloud.training/aws-developer-tools/ 