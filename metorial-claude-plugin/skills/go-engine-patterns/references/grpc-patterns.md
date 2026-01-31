# gRPC Patterns Reference

Deep patterns for gRPC implementation in the MCP engine. Load this reference for complex streaming, middleware, and connection management.

## Server Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      gRPC Server                            │
├─────────────────────────────────────────────────────────────┤
│  Interceptor Chain                                          │
│  ├── Recovery (panic → error)                               │
│  ├── Logging                                                │
│  ├── Authentication                                         │
│  └── Metrics                                                │
├─────────────────────────────────────────────────────────────┤
│  Service Implementations                                    │
│  ├── MCPEngine                                              │
│  ├── ConnectionManager                                      │
│  └── HealthCheck                                            │
├─────────────────────────────────────────────────────────────┤
│  Connection Pool                                            │
│  └── MCP Server Connections                                 │
└─────────────────────────────────────────────────────────────┘
```

## Server Setup

### Complete Server Configuration

```go
package main

import (
    "context"
    "net"
    "os"
    "os/signal"
    "syscall"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/health"
    healthpb "google.golang.org/grpc/health/grpc_health_v1"
    "google.golang.org/grpc/keepalive"
    "google.golang.org/grpc/reflection"
)

func main() {
    // Load config
    cfg, err := LoadConfig()
    if err != nil {
        log.Fatalf("failed to load config: %v", err)
    }

    // Create server with options
    srv := grpc.NewServer(
        grpc.ChainUnaryInterceptor(
            recoveryInterceptor(),
            loggingInterceptor(),
            authInterceptor(),
            metricsInterceptor(),
        ),
        grpc.ChainStreamInterceptor(
            streamRecoveryInterceptor(),
            streamLoggingInterceptor(),
            streamAuthInterceptor(),
        ),
        grpc.KeepaliveParams(keepalive.ServerParameters{
            MaxConnectionIdle:     15 * time.Minute,
            MaxConnectionAge:      30 * time.Minute,
            MaxConnectionAgeGrace: 5 * time.Minute,
            Time:                  5 * time.Minute,
            Timeout:               1 * time.Minute,
        }),
        grpc.KeepaliveEnforcementPolicy(keepalive.EnforcementPolicy{
            MinTime:             5 * time.Minute,
            PermitWithoutStream: true,
        }),
    )

    // Register services
    pb.RegisterMCPEngineServer(srv, NewEngineServer(cfg))

    // Health check
    healthSrv := health.NewServer()
    healthpb.RegisterHealthServer(srv, healthSrv)
    healthSrv.SetServingStatus("", healthpb.HealthCheckResponse_SERVING)

    // Reflection for debugging (disable in production)
    if cfg.EnableReflection {
        reflection.Register(srv)
    }

    // Start listener
    lis, err := net.Listen("tcp", cfg.Address())
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    // Graceful shutdown
    go func() {
        sigCh := make(chan os.Signal, 1)
        signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)
        <-sigCh

        healthSrv.SetServingStatus("", healthpb.HealthCheckResponse_NOT_SERVING)

        // Allow in-flight requests to complete
        time.Sleep(cfg.DrainPeriod)

        srv.GracefulStop()
    }()

    log.Printf("server listening on %s", cfg.Address())
    if err := srv.Serve(lis); err != nil {
        log.Fatalf("server error: %v", err)
    }
}
```

## Interceptors

### Unary Interceptors

```go
// Recovery interceptor
func recoveryInterceptor() grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (resp interface{}, err error) {
        defer func() {
            if r := recover(); r != nil {
                err = status.Errorf(codes.Internal, "panic: %v", r)
                // Log stack trace
                debug.PrintStack()
            }
        }()
        return handler(ctx, req)
    }
}

// Logging interceptor
func loggingInterceptor() grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        start := time.Now()

        resp, err := handler(ctx, req)

        duration := time.Since(start)
        code := status.Code(err)

        slog.InfoContext(ctx, "grpc call",
            slog.String("method", info.FullMethod),
            slog.String("code", code.String()),
            slog.Duration("duration", duration),
        )

        return resp, err
    }
}

// Auth interceptor
func authInterceptor() grpc.UnaryServerInterceptor {
    return func(
        ctx context.Context,
        req interface{},
        info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler,
    ) (interface{}, error) {
        // Skip auth for health checks
        if info.FullMethod == "/grpc.health.v1.Health/Check" {
            return handler(ctx, req)
        }

        md, ok := metadata.FromIncomingContext(ctx)
        if !ok {
            return nil, status.Error(codes.Unauthenticated, "missing metadata")
        }

        tokens := md.Get("authorization")
        if len(tokens) == 0 {
            return nil, status.Error(codes.Unauthenticated, "missing token")
        }

        // Validate token
        claims, err := validateToken(tokens[0])
        if err != nil {
            return nil, status.Error(codes.Unauthenticated, "invalid token")
        }

        // Add claims to context
        ctx = context.WithValue(ctx, claimsKey{}, claims)

        return handler(ctx, req)
    }
}
```

### Stream Interceptors

```go
// Wrapped stream for interceptors
type wrappedStream struct {
    grpc.ServerStream
    ctx context.Context
}

func (w *wrappedStream) Context() context.Context {
    return w.ctx
}

// Stream logging interceptor
func streamLoggingInterceptor() grpc.StreamServerInterceptor {
    return func(
        srv interface{},
        ss grpc.ServerStream,
        info *grpc.StreamServerInfo,
        handler grpc.StreamHandler,
    ) error {
        start := time.Now()

        err := handler(srv, ss)

        slog.InfoContext(ss.Context(), "grpc stream",
            slog.String("method", info.FullMethod),
            slog.String("code", status.Code(err).String()),
            slog.Duration("duration", time.Since(start)),
        )

        return err
    }
}

// Stream recovery interceptor
func streamRecoveryInterceptor() grpc.StreamServerInterceptor {
    return func(
        srv interface{},
        ss grpc.ServerStream,
        info *grpc.StreamServerInfo,
        handler grpc.StreamHandler,
    ) (err error) {
        defer func() {
            if r := recover(); r != nil {
                err = status.Errorf(codes.Internal, "panic: %v", r)
                debug.PrintStack()
            }
        }()
        return handler(srv, ss)
    }
}
```

## Bidirectional Streaming

```go
func (s *engineServer) BiDiStream(
    stream pb.MCPEngine_BiDiStreamServer,
) error {
    ctx := stream.Context()

    // Create channels for send/receive
    sendCh := make(chan *pb.Response, 100)
    errCh := make(chan error, 2)

    // Sender goroutine
    go func() {
        for {
            select {
            case <-ctx.Done():
                return
            case resp, ok := <-sendCh:
                if !ok {
                    return
                }
                if err := stream.Send(resp); err != nil {
                    errCh <- fmt.Errorf("send error: %w", err)
                    return
                }
            }
        }
    }()

    // Receiver loop
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            close(sendCh)
            return nil
        }
        if err != nil {
            return err
        }

        // Process request
        resp, err := s.processRequest(ctx, req)
        if err != nil {
            return err
        }

        select {
        case sendCh <- resp:
        case err := <-errCh:
            return err
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}
```

## Connection Management

### Connection Pool

```go
type ConnectionPool struct {
    mu          sync.RWMutex
    connections map[string]*Connection
    maxConns    int
    logger      *slog.Logger
}

type Connection struct {
    ID        string
    ServerID  string
    Status    ConnectionStatus
    Client    pb.MCPServerClient
    conn      *grpc.ClientConn
    createdAt time.Time
    lastUsed  time.Time
    mu        sync.Mutex
}

func NewConnectionPool(maxConns int, logger *slog.Logger) *ConnectionPool {
    return &ConnectionPool{
        connections: make(map[string]*Connection),
        maxConns:    maxConns,
        logger:      logger,
    }
}

func (p *ConnectionPool) Get(ctx context.Context, serverID string) (*Connection, error) {
    p.mu.RLock()
    for _, conn := range p.connections {
        if conn.ServerID == serverID && conn.Status == StatusConnected {
            conn.mu.Lock()
            conn.lastUsed = time.Now()
            conn.mu.Unlock()
            p.mu.RUnlock()
            return conn, nil
        }
    }
    p.mu.RUnlock()

    // Create new connection
    return p.create(ctx, serverID)
}

func (p *ConnectionPool) create(ctx context.Context, serverID string) (*Connection, error) {
    p.mu.Lock()
    defer p.mu.Unlock()

    // Check pool size
    if len(p.connections) >= p.maxConns {
        // Evict least recently used
        p.evictLRU()
    }

    // Dial server
    conn, err := grpc.DialContext(ctx, serverID,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithBlock(),
    )
    if err != nil {
        return nil, fmt.Errorf("dial %s: %w", serverID, err)
    }

    connection := &Connection{
        ID:        uuid.New().String(),
        ServerID:  serverID,
        Status:    StatusConnected,
        Client:    pb.NewMCPServerClient(conn),
        conn:      conn,
        createdAt: time.Now(),
        lastUsed:  time.Now(),
    }

    p.connections[connection.ID] = connection
    return connection, nil
}

func (p *ConnectionPool) evictLRU() {
    var oldest *Connection
    for _, conn := range p.connections {
        if oldest == nil || conn.lastUsed.Before(oldest.lastUsed) {
            oldest = conn
        }
    }
    if oldest != nil {
        oldest.conn.Close()
        delete(p.connections, oldest.ID)
    }
}
```

### Health Monitoring

```go
func (p *ConnectionPool) StartHealthCheck(ctx context.Context, interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            p.checkConnections(ctx)
        }
    }
}

func (p *ConnectionPool) checkConnections(ctx context.Context) {
    p.mu.RLock()
    conns := make([]*Connection, 0, len(p.connections))
    for _, conn := range p.connections {
        conns = append(conns, conn)
    }
    p.mu.RUnlock()

    for _, conn := range conns {
        state := conn.conn.GetState()
        if state == connectivity.TransientFailure || state == connectivity.Shutdown {
            p.logger.WarnContext(ctx, "unhealthy connection",
                slog.String("id", conn.ID),
                slog.String("server", conn.ServerID),
                slog.String("state", state.String()),
            )

            // Attempt reconnect or remove
            conn.mu.Lock()
            conn.Status = StatusDisconnected
            conn.mu.Unlock()
        }
    }
}
```

## Error Handling

### Error Mapping

```go
// Map internal errors to gRPC status codes
func toGRPCError(err error) error {
    if err == nil {
        return nil
    }

    switch {
    case errors.Is(err, context.Canceled):
        return status.Error(codes.Canceled, err.Error())
    case errors.Is(err, context.DeadlineExceeded):
        return status.Error(codes.DeadlineExceeded, err.Error())
    case errors.Is(err, ErrNotFound):
        return status.Error(codes.NotFound, err.Error())
    case errors.Is(err, ErrInvalidInput):
        return status.Error(codes.InvalidArgument, err.Error())
    case errors.Is(err, ErrUnauthorized):
        return status.Error(codes.Unauthenticated, err.Error())
    case errors.Is(err, ErrPermissionDenied):
        return status.Error(codes.PermissionDenied, err.Error())
    default:
        return status.Error(codes.Internal, err.Error())
    }
}

// Extract error details
func withDetails(err error, details ...proto.Message) error {
    st, ok := status.FromError(err)
    if !ok {
        st = status.New(codes.Internal, err.Error())
    }

    stWithDetails, err := st.WithDetails(details...)
    if err != nil {
        return st.Err()
    }

    return stWithDetails.Err()
}
```

## Metadata and Context

```go
// Extract request ID from context
func requestIDFromContext(ctx context.Context) string {
    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return uuid.New().String()
    }

    ids := md.Get("x-request-id")
    if len(ids) > 0 {
        return ids[0]
    }

    return uuid.New().String()
}

// Add metadata to outgoing context
func withRequestID(ctx context.Context, requestID string) context.Context {
    md := metadata.Pairs("x-request-id", requestID)
    return metadata.NewOutgoingContext(ctx, md)
}

// Propagate metadata through calls
func (s *engineServer) CallDownstream(ctx context.Context, req *pb.Request) (*pb.Response, error) {
    // Extract and propagate metadata
    inMD, _ := metadata.FromIncomingContext(ctx)
    outCtx := metadata.NewOutgoingContext(ctx, inMD)

    return s.downstream.Call(outCtx, req)
}
```

## Testing gRPC

### In-Memory Server

```go
func setupTestServer(t *testing.T) (*grpc.Server, string) {
    t.Helper()

    // Use bufconn for in-memory connection
    lis := bufconn.Listen(1024 * 1024)

    srv := grpc.NewServer()
    pb.RegisterMCPEngineServer(srv, NewEngineServer(testConfig))

    go func() {
        if err := srv.Serve(lis); err != nil && err != grpc.ErrServerStopped {
            t.Errorf("server error: %v", err)
        }
    }()

    t.Cleanup(func() {
        srv.GracefulStop()
    })

    return srv, lis.Addr().String()
}

func testClient(t *testing.T, lis *bufconn.Listener) pb.MCPEngineClient {
    t.Helper()

    conn, err := grpc.DialContext(context.Background(), "",
        grpc.WithContextDialer(func(ctx context.Context, _ string) (net.Conn, error) {
            return lis.DialContext(ctx)
        }),
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    require.NoError(t, err)

    t.Cleanup(func() {
        conn.Close()
    })

    return pb.NewMCPEngineClient(conn)
}
```

### Stream Testing

```go
func TestStream(t *testing.T) {
    _, lis := setupTestServer(t)
    client := testClient(t, lis)

    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    stream, err := client.Stream(ctx, &pb.StreamRequest{
        ConnectionId: "test-conn",
    })
    require.NoError(t, err)

    // Receive messages
    var messages []*pb.StreamResponse
    for {
        msg, err := stream.Recv()
        if err == io.EOF {
            break
        }
        require.NoError(t, err)
        messages = append(messages, msg)

        if len(messages) >= 10 {
            break
        }
    }

    assert.NotEmpty(t, messages)
}
```

---

**Note**: For the most current patterns, use `drift_context` to analyze the actual engine codebase.
