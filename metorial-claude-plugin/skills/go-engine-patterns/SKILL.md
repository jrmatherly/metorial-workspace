---
name: Go Engine Patterns
description: This skill should be used when the user asks to "modify engine", "Go code", "gRPC", "mcp-engine", "golangci-lint", "protobuf", mentions "metorial-platform/src/mcp-engine/", or discusses Go development for the Metorial MCP engine. Provides guidance for Go development following engine conventions including golangci-lint v2 compliance.
version: 1.0.0
---

# Go Engine Patterns

This skill provides guidance for developing the Go-based MCP engine in `metorial-platform/src/mcp-engine/`. The engine handles MCP server connections, streaming, and protocol implementation.

## Context Loading

Before modifying the engine, gather context from existing tools:

### 1. Pattern Context (Drift)

Use Drift to understand existing patterns:

```
drift_context targeting metorial-platform/src/mcp-engine/
```

This provides detected patterns for:
- Error handling idioms
- Context propagation
- gRPC patterns
- Concurrency patterns

### 2. Workflow Context (Serena)

Use Serena for development workflow:

```
read_memory('development_workflow')
```

Key commands:
- `mise run engine:dev` - Development with hot reload
- `mise run engine:build` - Build binary
- `mise run engine:test` - Run tests
- `mise run engine:lint` - Linting with golangci-lint v2
- `mise run engine:proto` - Generate protobuf

### 3. Code Navigation (Serena)

Navigate existing code:

```
find_symbol targeting metorial-platform/src/mcp-engine/
```

## golangci-lint v2 Configuration

**Critical**: The engine uses golangci-lint v2. Key differences from v1:

```yaml
# .golangci.yml
version: "2"  # REQUIRED for v2

linters:
  enable:
    - errcheck
    - govet
    - staticcheck
    - unused
    - gosec
    - revive

  settings:
    errcheck:
      check-type-assertions: true
      check-blank: true
    govet:
      enable-all: true
    revive:
      rules:
        - name: exported
          arguments:
            - "checkPrivateReceivers"
            - "sayRepetitiveInsteadOfStutters"

  exclusions:
    paths:
      - ".*_test\\.go$"
      - "internal/mock/"
```

**v2 Changes:**
- `linters-settings` → `linters.settings`
- `issues.exclude-dirs` → `linters.exclusions.paths`
- `typecheck` removed (not a linter in v2)
- `gosimple` merged into `staticcheck`

## Project Structure

```
mcp-engine/
├── cmd/
│   └── engine/
│       └── main.go        # Entry point
├── internal/
│   ├── server/            # gRPC server
│   ├── mcp/               # MCP protocol
│   ├── connection/        # Connection management
│   └── stream/            # Streaming handlers
├── pkg/
│   └── proto/             # Generated protobuf
├── api/
│   └── proto/             # Protobuf definitions
├── .golangci.yml          # Linter config (v2)
├── go.mod
└── Makefile
```

## Go Idioms

### Error Handling

Follow Go conventions for error handling:

```go
// Always check errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doing something: %w", err)
}

// Wrap errors with context
if err := processItem(item); err != nil {
    return fmt.Errorf("processing item %s: %w", item.ID, err)
}

// Define sentinel errors for known conditions
var (
    ErrNotFound      = errors.New("not found")
    ErrUnauthorized  = errors.New("unauthorized")
    ErrInvalidInput  = errors.New("invalid input")
)

// Check for specific errors
if errors.Is(err, ErrNotFound) {
    // Handle not found
}
```

### Context Propagation

Always propagate context:

```go
func (s *Server) HandleRequest(ctx context.Context, req *Request) (*Response, error) {
    // Check for cancellation
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }

    // Pass context to all operations
    result, err := s.store.Get(ctx, req.ID)
    if err != nil {
        return nil, err
    }

    return &Response{Data: result}, nil
}
```

### Concurrency Patterns

Use goroutines safely:

```go
// Use errgroup for concurrent operations
func (s *Server) ProcessItems(ctx context.Context, items []Item) error {
    g, ctx := errgroup.WithContext(ctx)

    for _, item := range items {
        item := item // Capture for goroutine
        g.Go(func() error {
            return s.processItem(ctx, item)
        })
    }

    return g.Wait()
}

// Use channels for communication
func (s *Server) StreamResults(ctx context.Context) <-chan Result {
    ch := make(chan Result)

    go func() {
        defer close(ch)
        for {
            select {
            case <-ctx.Done():
                return
            case result := <-s.results:
                ch <- result
            }
        }
    }()

    return ch
}
```

### Resource Cleanup

Use defer for cleanup:

```go
func (c *Connection) Execute(ctx context.Context) error {
    c.mu.Lock()
    defer c.mu.Unlock()

    conn, err := c.pool.Get(ctx)
    if err != nil {
        return err
    }
    defer conn.Close()

    return conn.Execute(ctx)
}
```

## gRPC Patterns

### Service Definition

```protobuf
// api/proto/engine.proto
syntax = "proto3";

package engine.v1;

option go_package = "github.com/metorial/mcp-engine/pkg/proto/engine/v1";

service MCPEngine {
    rpc Connect(ConnectRequest) returns (ConnectResponse);
    rpc Execute(ExecuteRequest) returns (ExecuteResponse);
    rpc Stream(StreamRequest) returns (stream StreamResponse);
}

message ConnectRequest {
    string server_id = 1;
    map<string, string> config = 2;
}
```

### Server Implementation

```go
type engineServer struct {
    pb.UnimplementedMCPEngineServer
    connManager *connection.Manager
    logger      *slog.Logger
}

func NewServer(cm *connection.Manager, logger *slog.Logger) pb.MCPEngineServer {
    return &engineServer{
        connManager: cm,
        logger:      logger,
    }
}

func (s *engineServer) Connect(
    ctx context.Context,
    req *pb.ConnectRequest,
) (*pb.ConnectResponse, error) {
    s.logger.InfoContext(ctx, "connecting to server",
        slog.String("server_id", req.ServerId))

    conn, err := s.connManager.Connect(ctx, req.ServerId, req.Config)
    if err != nil {
        return nil, status.Errorf(codes.Internal, "connect failed: %v", err)
    }

    return &pb.ConnectResponse{
        ConnectionId: conn.ID,
        Status:       pb.Status_CONNECTED,
    }, nil
}
```

### Streaming

```go
func (s *engineServer) Stream(
    req *pb.StreamRequest,
    stream pb.MCPEngine_StreamServer,
) error {
    ctx := stream.Context()

    ch, err := s.connManager.Subscribe(ctx, req.ConnectionId)
    if err != nil {
        return status.Errorf(codes.NotFound, "connection not found")
    }

    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case msg, ok := <-ch:
            if !ok {
                return nil
            }
            if err := stream.Send(msg); err != nil {
                return err
            }
        }
    }
}
```

## Structured Logging

Use slog for structured logging:

```go
import "log/slog"

// Create logger with context
logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))

// Log with context
logger.InfoContext(ctx, "processing request",
    slog.String("request_id", requestID),
    slog.Int("item_count", len(items)),
)

// Log errors
if err != nil {
    logger.ErrorContext(ctx, "operation failed",
        slog.String("operation", "connect"),
        slog.Any("error", err),
    )
}

// Add context to logger
logger = logger.With(
    slog.String("service", "mcp-engine"),
    slog.String("version", version),
)
```

## Testing Patterns

### Table-Driven Tests

```go
func TestParser(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    *Result
        wantErr bool
    }{
        {
            name:  "valid input",
            input: "test",
            want:  &Result{Value: "test"},
        },
        {
            name:    "empty input",
            input:   "",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Parse(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Parse() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("Parse() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

### Mocking

```go
//go:generate mockgen -source=store.go -destination=mock/store_mock.go -package=mock

type Store interface {
    Get(ctx context.Context, id string) (*Item, error)
    Put(ctx context.Context, item *Item) error
}

// In tests
func TestHandler(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockStore := mock.NewMockStore(ctrl)
    mockStore.EXPECT().
        Get(gomock.Any(), "test-id").
        Return(&Item{ID: "test-id"}, nil)

    handler := NewHandler(mockStore)
    // Test handler...
}
```

### Integration Tests

```go
// +build integration

func TestIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Setup test server
    srv := setupTestServer(t)
    defer srv.Close()

    // Run integration tests
    client := NewClient(srv.Addr())
    resp, err := client.Connect(ctx, &ConnectRequest{...})
    require.NoError(t, err)
    assert.Equal(t, StatusConnected, resp.Status)
}
```

## Configuration Pattern

```go
type Config struct {
    Host        string        `env:"HOST" default:"0.0.0.0"`
    Port        int           `env:"PORT" default:"50051"`
    LogLevel    string        `env:"LOG_LEVEL" default:"info"`
    GracePeriod time.Duration `env:"GRACE_PERIOD" default:"30s"`
}

func LoadConfig() (*Config, error) {
    cfg := &Config{}

    if err := envconfig.Process("ENGINE", cfg); err != nil {
        return nil, fmt.Errorf("loading config: %w", err)
    }

    return cfg, nil
}
```

## Graceful Shutdown

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Setup signal handling
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)

    // Start server
    srv := NewServer(cfg)
    go func() {
        if err := srv.Start(ctx); err != nil {
            log.Fatalf("server error: %v", err)
        }
    }()

    // Wait for shutdown signal
    <-sigCh
    log.Println("shutting down...")

    // Graceful shutdown with timeout
    shutdownCtx, shutdownCancel := context.WithTimeout(
        context.Background(),
        cfg.GracePeriod,
    )
    defer shutdownCancel()

    if err := srv.Shutdown(shutdownCtx); err != nil {
        log.Printf("shutdown error: %v", err)
    }
}
```

## Validation Checklist

Before submitting engine changes:

- [ ] Code passes `mise run engine:lint` (golangci-lint v2)
- [ ] All tests pass `mise run engine:test`
- [ ] Context propagated through all functions
- [ ] Errors wrapped with context
- [ ] Resources cleaned up with defer
- [ ] Protobuf regenerated if .proto changed
- [ ] Pattern compliance verified (`drift_validate_change`)

## Additional Resources

### Reference Files

For detailed patterns:

- **`references/grpc-patterns.md`** - Deep gRPC implementation patterns

### External References

- Serena memory: `development_workflow` for build/test commands
- Drift patterns: `drift_context` for existing conventions
- Engine source: `metorial-platform/src/mcp-engine/`
