# gRPC 使用

标签（空格分隔）： rpc

---

[toc]

> 详见 [gRPC](https://www.grpc.io/docs/) 官方文档。

## 概述
> 可以一次性的在一个 `.proto` 文件中定义服务并使用任何支持它的语言去实现客户端和服务器。

### 使用步骤

1. 在一个 `.proto` 文件内**定义服务**
2. 用 `protocol buffer` 编译器生成**服务器和客户端代码**
3. 使用 `gRPC` 的相关 `API` （go、java等）为服务实现**客户端**和**服务器**

### 优点

- 解决了不同语言及环境间通信的复杂性
- 高效的序列号
- 简单的 IDL
- 容易进行接口更新

## go 使用 grpc 示例

### 定义服务
> 在 `.proto` 文件中，使用 `protocol buffers` 定义 gRPC `service` 、 `request` 以及 `response` 的类型。

```
// 定义服务，RouteGuide
service RouteGuide {
   ...
}
```

#### service 方法
> 在服务中定义 `rpc` 方法，指定请求的和响应类型。
> gRPC 允许定义4种类型的 service 方法，可在服务中使用。

1. **简单 RPC** ，客户端使用存根发送请求到服务器并等待响应返回。
2. **服务器端流式 RPC** ，客户端发送请求到服务器，拿到一个流去读取返回的消息序列。客户端读取返回的流，直到里面没有任何消息。
3. **客户端流式 RPC** ，客户端写入一个消息序列并将其发送到服务器，同样也是使用流。一旦客户端完成写入消息，它等待服务器完成读取返回它的响应。
4. **双向流式 RPC** ，双方使用读写流去发送一个消息序列。两个流独立操作，因此客户端和服务器可以以任意喜欢的顺序读写。

```
// 简单 RPC
rpc GetFeature(Point) returns (Feature) {}
// 服务器端流式 RPC
rpc ListFeatures(Rectangle) returns (stream Feature) {}
// 客户端流式 RPC
rpc RecordRoute(stream Point) returns (RouteSummary) {}
// 双向流式 RPC
rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
```

#### 消息类型
>  `.proto` 文件包含了所有请求的 `protocol buffer` 消息类型定义以及在服务方法中使用的响
应类型。

```
// Point 消息类型
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

### 生成客户端和服务器端代码
> 从 `.proto` 的服务定义中生成 gRPC 客户端和服务器端的接口（生成`.pb.go`文件）。

- 使用`protoc`生成`.pb.go`文件

    ```
    $ protoc --go_out=plugins=grpc:. *.proto
    ```

- 包含内容
    - 所有用于填充，序列化和获取请求和响应消息类型的 `protocol buffer` 代码
    - 为客户端调用，定义在`RouteGuide`服务方法的接口类型（或者 **存根** ）
    - 为服务器使用，定义在`RouteGuide`服务方法去实现的接口类型（或者 **存根** ）

### 创建服务器

- 实现通过**服务定义**生成的服务接口
- 运行一个 gRPC 服务器，监听来自客户端的请求并返回服务的响应

#### 实现 RouteGuide

- 简单 RPC

    ```
    func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
        for _, feature := range s.savedFeatures {
            if proto.Equal(feature.Location, point) {
                return feature, nil
            }
        }
        // No feature was found, return an unnamed feature
        return &pb.Feature{"", point}, nil
    }
    ```
    
- 服务器端流式 RPC

    ```
    func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
        for _, feature := range s.savedFeatures {
            if inRange(feature.Location, rect) {
                if err := stream.Send(feature); err != nil {
                    return err
                }
            }
        }
        return nil
    }
    ```
    
- 客户端流式 RPC

    ```
    func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
        var pointCount, featureCount, distance int32
        var lastPoint *pb.Point
        startTime := time.Now()
        for {
            point, err := stream.Recv()
            if err == io.EOF {
                endTime := time.Now()
                return stream.SendAndClose(&pb.RouteSummary{
                    PointCount:   pointCount,
                    FeatureCount: featureCount,
                    Distance:     distance,
                    ElapsedTime:  int32(endTime.Sub(startTime).Seconds()),
                })
            }
            if err != nil {
                return err
            }
            pointCount++
            for _, feature := range s.savedFeatures {
                if proto.Equal(feature.Location, point) {
                    featureCount++
                }
            }
            if lastPoint != nil {
                distance += calcDistance(lastPoint, point)
            }
            lastPoint = point
        }
    }
    ```

- 双向流式 RPC

    ```
    func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
        for {
            in, err := stream.Recv()
            if err == io.EOF {
                return nil
            }
            if err != nil {
                return err
            }
            key := serialize(in.Location)
                    ... // look for notes to be sent to client
            for _, note := range s.routeNotes[key] {
                if err := stream.Send(note); err != nil {
                    return err
                }
            }
        }
    }
    ```

#### 启动服务器

```
flag.Parse()
lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
if err != nil {
        log.Fatalf("failed to listen: %v", err)
}
grpcServer := grpc.NewServer()
pb.RegisterRouteGuideServer(grpcServer, &routeGuideServer{})
... // determine whether to use TLS
grpcServer.Serve(lis)
```

1. 使用 `lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))` 指定客户端请求的监听端口。
1. 使用`grpc.NewServer()`创建 gRPC 服务器的一个实例。
1. 在 gRPC 服务器注册**服务实现**。
1. 用服务器 `Serve()` 方法以及端口信息区实现阻塞等待，直到进程被杀死或者 `Stop()` 被调用。

### 创建客户端

#### 创建存根
> 为了调用服务方法，通过给 `grpc.Dial()` 传入服务器地址和端口号，创建一个 `gRPC channel` 和服务器交互。

```
// 创建 gRPC channel
// 可以使用 DialOptions 在 grpc.Dial 中设置授权认证（如， TLS，GCE认证，JWT认证）
conn, err := grpc.Dial(*serverAddr)
if err != nil {
    ...
}
defer conn.Close()

// 执行 RPC
// pb包中 NewRouteGuideClient 方法通过 .proto 生成
client := pb.NewRouteGuideClient(conn)
```

#### 调用服务方法

- 简单 RPC

    ```
    feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
    if err != nil {
            ...
    }
    ```

- 服务器端流式 RPC

    ```
    rect := &pb.Rectangle{ ... }  // initialize a pb.Rectangle
    stream, err := client.ListFeatures(context.Background(), rect)
    if err != nil {
        ...
    }
    for {
        feature, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
        }
        log.Println(feature)
    }
    ```

- 客户端流式 RPC

    ```
    // Create a random number of random points
    r := rand.New(rand.NewSource(time.Now().UnixNano()))
    pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points
    var points []*pb.Point
    for i := 0; i < pointCount; i++ {
        points = append(points, randomPoint(r))
    }
    log.Printf("Traversing %d points.", len(points))
    stream, err := client.RecordRoute(context.Background())
    if err != nil {
        log.Fatalf("%v.RecordRoute(_) = _, %v", client, err)
    }
    for _, point := range points {
        if err := stream.Send(point); err != nil {
            log.Fatalf("%v.Send(%v) = %v", stream, point, err)
        }
    }
    reply, err := stream.CloseAndRecv()
    if err != nil {
        log.Fatalf("%v.CloseAndRecv() got error %v, want %v", stream, err, nil)
    }
    log.Printf("Route summary: %v", reply)
    ```

- 双向流式 RPC

    ```
    stream, err := client.RouteChat(context.Background())
    waitc := make(chan struct{})
    go func() {
        for {
            in, err := stream.Recv()
            if err == io.EOF {
                // read done.
                close(waitc)
                return
            }
            if err != nil {
                log.Fatalf("Failed to receive a note : %v", err)
            }
            log.Printf("Got message %s at point(%d, %d)", in.Message, in.Location.Latitude, in.Location.Longitude)
        }
    }()
    for _, note := range notes {
        if err := stream.Send(note); err != nil {
            log.Fatalf("Failed to send a note: %v", err)
        }
    }
    stream.CloseSend()
    <-waitc
    ```
