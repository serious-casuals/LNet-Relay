---
layout: default
title: API Reference
parent: LNet Library
nav_order: 2
---

# LNet API Reference

Complete reference for all classes, methods, and types in the LNet library.

## Core Classes

### Client Class

The main entry point for LNET connections with keepalive support and event-driven architecture.

#### Constructor & Factory Methods

```csharp
public static async ValueTask<Client> CreateAsync(
    Options options, 
    ILoggerFactory? loggerFactory = null, 
    CancellationToken cancellationToken = default)
```

Creates and connects a new LNET client.

**Parameters:**
- `options`: Connection configuration
- `loggerFactory`: Optional logger factory (creates default if null)
- `cancellationToken`: Cancellation token for connection timeout

**Returns:** Connected client instance

**Throws:**
- `ArgumentNullException`: If options is null
- `InvalidOperationException`: If connection fails
- `TimeoutException`: If connection times out

#### Properties

```csharp
public bool IsConnected { get; }
```
Indicates if the client is currently connected.

```csharp
public string[] SupportedMessageTypes { get; }
```
Gets the supported message types from the parser.

```csharp
public TimeSpan? Uptime { get; }
```
Gets the connection uptime since connection established.

#### Events

```csharp
public event Func<ValueTask>? OnConnected;
```
Fired when the client successfully connects to the server.

```csharp
public event Func<ValueTask>? OnDisconnected;
```
Fired when the client disconnects from the server.

```csharp
public event Func<Exception, ValueTask>? OnError;
```
Fired when an error occurs during operation.

```csharp
public event Func<ILNetMessage, ValueTask>? OnMessageReceived;
```
Fired when a message is received (before processing).

#### Message Processing Methods

```csharp
public async ValueTask<ILNetMessage?> ReceiveAsync(
    CancellationToken cancellationToken = default)
```

Receive the next message from the server.

**Returns:** The received message object, or null if connection closed

```csharp
public async IAsyncEnumerable<ILNetMessage> GetMessageStreamAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
```

Get an async enumerable of all messages for continuous processing.

**Returns:** Async enumerable of messages

#### Sending Methods

```csharp
public async ValueTask SendMessageAsync(
    Message message, 
    CancellationToken cancellationToken = default)
```

Send a message to the server.

**Parameters:**
- `message`: Message object to send
- `cancellationToken`: Cancellation token

```csharp
public async ValueTask<int> JoinChannelAsync(
    string channel, 
    CancellationToken cancellationToken = default)
```

Join a channel.

**Parameters:**
- `channel`: Channel name to join
- `cancellationToken`: Cancellation token

**Returns:** Number of bytes sent

```csharp
public async ValueTask<int> LeaveChannelAsync(
    string channel, 
    CancellationToken cancellationToken = default)
```

Leave a channel.

**Parameters:**
- `channel`: Channel name to leave
- `cancellationToken`: Cancellation token

**Returns:** Number of bytes sent

```csharp
public async ValueTask SendPingAsync(
    CancellationToken cancellationToken = default)
```

Send a manual ping message.

```csharp
public async ValueTask SendPongAsync(
    CancellationToken cancellationToken = default)
```

Send a pong response (typically in response to a ping).

```csharp
public async ValueTask<int> SendLocationRequestAsync(
    string targetUser, 
    string requestType = "locate", 
    CancellationToken cancellationToken = default)
```

Send a location request to another user.

**Parameters:**
- `targetUser`: User to request location from
- `requestType`: Type of request (default: "locate")
- `cancellationToken`: Cancellation token

**Returns:** Number of bytes sent

```csharp
public async ValueTask<int> SendStatusAsync(
    string status, 
    string? statusText = null, 
    CancellationToken cancellationToken = default)
```

Send a status message.

**Parameters:**
- `status`: Status to set
- `statusText`: Optional status text
- `cancellationToken`: Cancellation token

**Returns:** Number of bytes sent

#### Keepalive Methods

```csharp
public void StartKeepalive()
```
Manually start keepalive timer if it was disabled in options.

```csharp
public void StopKeepalive()
```
Stop the keepalive timer.

```csharp
public void SetKeepaliveInterval(int seconds)
```
Change the keepalive interval. Restarts the timer with new interval.

**Parameters:**
- `seconds`: New interval in seconds (must be > 0)

#### Statistics & Management

```csharp
public ClientStatistics GetStatistics()
```
Gets comprehensive client statistics.

**Returns:** Client statistics structure

```csharp
public void Close()
```
Close the connection (alias for Dispose).

## Configuration Classes

### Options Class

Configuration options for LNET client connections.

```csharp
public class Options
{
    public string Host { get; set; } = "lnet.lichproject.org";
    public string User { get; set; } = string.Empty;
    public string Game { get; set; } = "DR";
    public string Password { get; set; } = string.Empty;
    public bool Debug { get; set; } = false;
    public bool EnableFileLogging { get; set; } = false;
    public string? LogFilePath { get; set; }
    public bool DenyAllRequests { get; set; } = false;
    public RemoteCertificateValidationCallback? CertificateValidationCallback { get; set; }
    public bool AcceptAnyCertificate { get; set; } = false;
    public bool EnableKeepalive { get; set; } = true;
    public int KeepaliveIntervalSeconds { get; set; } = 60;
    public int ConnectionTimeoutSeconds { get; set; } = 30;
    public int MaxMessageQueueSize { get; set; } = 1000;
    public XmlParserType ParserType { get; set; } = XmlParserType.Auto;
}
```

**Properties:**
- `Host`: Server hostname and optional port (default: "lnet.lichproject.org")
- `User`: User specifies what user to authenticate to the remote server
- `Game`: Game this user is from ("DR" for DragonRealms, "GSIV" for GemStone IV)
- `Password`: Optional password (not currently implemented by LNET protocol)
- `Debug`: Enable debug-level logging
- `EnableFileLogging`: Enable file logging using ZLogger
- `LogFilePath`: Custom log file path (if null, uses default location)
- `DenyAllRequests`: Deny all incoming requests automatically
- `CertificateValidationCallback`: Custom certificate validation callback for SSL
- `AcceptAnyCertificate`: Accept any SSL certificate without validation (use with caution)
- `EnableKeepalive`: Enable automatic keepalive pings (default: true)
- `KeepaliveIntervalSeconds`: Keepalive interval in seconds (default: 60)
- `ConnectionTimeoutSeconds`: Connection timeout in seconds (default: 30)
- `MaxMessageQueueSize`: Maximum message queue size before blocking (default: 1000)
- `ParserType`: XML parser type to use (default: Auto)

### Constants Class

LNET protocol constants.

```csharp
public static class Constants
{
    public const string ClientVersion = "1.6";
    public const string LichVersion = "4.6.58";
    public const int DefaultPort = 7155;
}
```

## Message Types

All messages implement the `ILNetMessage` interface.

### ILNetMessage Interface

```csharp
public interface ILNetMessage
{
    string MessageType { get; }
    DateTime ReceivedAt { get; }
}
```

### Message Struct

Represents an LNET message with various attributes.

```csharp
public readonly struct Message : ILNetMessage
{
    public readonly string? Type;
    public readonly string? From;
    public readonly string? To;
    public readonly string? Subscription;
    public readonly string? Channel;
    public readonly string? Text;
    public string MessageType { get; }
    public DateTime ReceivedAt { get; }
}
```

**Constructor:**
```csharp
public Message(string? type = null, string? from = null, string? to = null,
              string? subscription = null, string? channel = null, string? text = null,
              DateTime? receivedAt = null)
```

### Chat Message Types

```csharp
public readonly struct Chat : ILNetMessage
{
    public readonly string? Type;
    public readonly string? User;
    public readonly string? Channel;
    public readonly string? Text;
}
```

```csharp
public readonly struct PrivateMessage : ILNetMessage
{
    public readonly string? User;
    public readonly string? Text;
}
```

```csharp
public readonly struct ChannelMessage : ILNetMessage
{
    public readonly string? User;
    public readonly string? Channel;
    public readonly string? Text;
}
```

### System Message Types

```csharp
public readonly struct Ping : ILNetMessage
{
    public string MessageType => "ping";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct Pong : ILNetMessage
{
    public string MessageType => "pong";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct LocationRequest : ILNetMessage
{
    public readonly string? User;
    public readonly string? RequestType;
    public string MessageType => "request";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct StatusMessage : ILNetMessage
{
    public readonly string? User;
    public readonly string? Status;
    public readonly string? Timestamp;
    public readonly string? StatusText;
    public string MessageType => "status";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct RoomMessage : ILNetMessage
{
    public readonly string? Action; // "join" or "leave"
    public readonly string? User;
    public readonly string? Room;
    public readonly string? Timestamp;
    public string MessageType => "room";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct AnnouncementMessage : ILNetMessage
{
    public readonly string? Priority;
    public readonly string? From;
    public readonly string? Timestamp;
    public readonly string? Text;
    public string MessageType => "announcement";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct AuthMessage : ILNetMessage
{
    public readonly string? Action; // "login", "logout", "challenge"
    public readonly string? User;
    public readonly string? Token;
    public readonly string? Result; // "success", "failed"
    public string MessageType => "auth";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct ConnectionStatus : ILNetMessage
{
    public readonly bool IsConnected;
    public readonly string? ServerHost;
    public readonly string? User;
    public readonly DateTime? ConnectedAt;
    public readonly TimeSpan? Uptime;
    public string MessageType => "connection";
    public DateTime ReceivedAt { get; }
}
```

```csharp
public readonly struct UnknownMessage : ILNetMessage
{
    public readonly string ElementName;
    public readonly string? RawXml;
    public string MessageType => "unknown";
    public DateTime ReceivedAt { get; }
}
```

## Parser Classes

### XmlParserType Enum

```csharp
public enum XmlParserType
{
    XmlReader,    // Traditional XmlReader-based parser
    SpanBased,    // Custom Span-based parser (high performance)
    Auto          // Automatically choose the best parser
}
```

### ILNetXmlParser Interface

```csharp
public interface ILNetXmlParser : IDisposable
{
    ValueTask<ILNetMessage?> ReadNextAsync(CancellationToken cancellationToken = default);
    string[] GetSupportedElements();
    object GetStatistics();
    void ValidateParser();
    void ValidateSourceGenerator();
    string ParserType { get; }
}
```

### XmlParserFactory Class

```csharp
public static class XmlParserFactory
{
    public static ILNetXmlParser CreateParser(XmlParserType parserType, Stream stream, ILoggerFactory loggerFactory);
    public static ILNetXmlParser CreateAutoParser(Stream stream, ILoggerFactory loggerFactory);
    public static XmlParserType GetRecommendedParserType();
}
```

## Statistics Classes

### ClientStatistics Struct

```csharp
public readonly struct ClientStatistics
{
    public bool IsConnected { get; init; }
    public string[] SupportedMessageTypes { get; init; }
    public bool KeepaliveEnabled { get; init; }
    public int KeepaliveIntervalSeconds { get; init; }
    public object? ParserStatistics { get; init; }
    public string ParserType { get; init; }
    public string Host { get; init; }
    public string User { get; init; }
    public string Game { get; init; }
    public DateTime? ConnectedAt { get; init; }
    public TimeSpan? Uptime { get; init; }
    public int TotalSupportedMessageTypes { get; }
}
```

### SpanParserStatistics Struct

```csharp
public readonly struct SpanParserStatistics
{
    public int BufferSize { get; init; }
    public bool IsSpanParserActive { get; init; }
    public string[] SupportedElements { get; init; }
    public string ParsingMode { get; init; }
    public long MessagesProcessed { get; init; }
    public long ElementsSkipped { get; init; }
    public long BytesProcessed { get; init; }
    public long ParseErrors { get; init; }
    public long BufferResizes { get; init; }
    public int TotalSupportedElements { get; }
    public double BytesPerMessage { get; }
}
```

### ParserStatistics Struct

```csharp
public readonly struct ParserStatistics
{
    public int BufferSize { get; init; }
    public bool IsSourceGeneratorActive { get; init; }
    public string[] SupportedElements { get; init; }
    public string ParsingMode { get; init; }
    public long MessagesProcessed { get; init; }
    public long ElementsSkipped { get; init; }
    public int TotalSupportedElements { get; }
}
```

## Logging Classes

### LNetLogging Class

High-performance logging configuration using ZLogger.

```csharp
public static class LNetLogging
{
    public static ILoggerFactory CreateLoggerFactory(
        bool enableDebug = false,
        bool enableConsole = true,
        bool enableFile = false,
        string? logFilePath = null);
        
    public static ILogger<T> CreateLogger<T>(bool enableDebug = false, bool enableFile = false);
    
    public static ILogger CreateLogger(string categoryName, bool enableDebug = false, bool enableFile = false);
}
```

### LNetLogEvents Class

Structured log event IDs for consistent logging.

```csharp
public static class LNetLogEvents
{
    // Connection events (1000-1999)
    public static readonly EventId Connected = new(1001, "Connected");
    public static readonly EventId Disconnected = new(1002, "Disconnected");
    public static readonly EventId ConnectionFailed = new(1003, "ConnectionFailed");
    public static readonly EventId LoginSent = new(1004, "LoginSent");
    public static readonly EventId SslHandshakeCompleted = new(1006, "SslHandshakeCompleted");

    // Message events (2000-2999)
    public static readonly EventId MessageReceived = new(2001, "MessageReceived");
    public static readonly EventId MessageSent = new(2002, "MessageSent");
    public static readonly EventId MessageQueueFull = new(2005, "MessageQueueFull");

    // Keepalive events (3000-3999)
    public static readonly EventId KeepalivePingSent = new(3001, "KeepalivePingSent");
    public static readonly EventId KeepalivePongReceived = new(3002, "KeepalivePongReceived");
    public static readonly EventId KeepaliveStarted = new(3003, "KeepaliveStarted");
    public static readonly EventId KeepaliveStopped = new(3004, "KeepaliveStopped");

    // Channel events (4000-4999)
    public static readonly EventId ChannelJoined = new(4001, "ChannelJoined");
    public static readonly EventId ChannelLeft = new(4002, "ChannelLeft");

    // Parser events (5000-5999)
    public static readonly EventId ParserInitialized = new(5001, "ParserInitialized");
    public static readonly EventId ParserValidated = new(5002, "ParserValidated");
    public static readonly EventId ElementSkipped = new(5003, "ElementSkipped");

    // Error events (9000-9999)
    public static readonly EventId GeneralError = new(9001, "GeneralError");
    public static readonly EventId NetworkError = new(9002, "NetworkError");
    public static readonly EventId ParseError = new(9003, "ParseError");
}
```

## Utility Classes

### DebugStream Class

Debug stream wrapper for logging network traffic.

```csharp
internal class DebugStream : Stream
{
    public long TotalBytesRead { get; }
    public long TotalBytesWritten { get; }
    public DebugStreamStatistics GetStatistics();
}
```

### DebugStreamStatistics Struct

```csharp
public readonly struct DebugStreamStatistics
{
    public long TotalBytesRead { get; init; }
    public long TotalBytesWritten { get; init; }
    public string InnerStreamType { get; init; }
    public bool CanRead { get; init; }
    public bool CanWrite { get; init; }
    public bool CanSeek { get; init; }
    public long Position { get; init; }
    public long Length { get; init; }
    public long TotalBytesTransferred { get; }
}
```

## Extension Methods

### DebugStreamExtensions Class

```csharp
public static class DebugStreamExtensions
{
    public static Stream WrapWithDebugIfEnabled(this Stream stream, ILogger logger, bool enableDebug);
    public static DebugStreamStatistics? GetDebugStatistics(this Stream stream);
}
```

## Error Handling

### Common Exceptions

The library may throw the following exceptions:

- **ArgumentNullException**: When required parameters are null
- **ArgumentException**: When configuration is invalid
- **InvalidOperationException**: When operations are called in invalid states
- **TimeoutException**: When connection or read operations timeout
- **SocketException**: When network issues occur
- **XmlException**: When XML parsing fails (handled internally with recovery)
- **AuthenticationException**: When SSL/TLS authentication fails
- **OutOfMemoryException**: When buffer allocation fails under memory pressure

### Event Error Handling

All events use `ValueTask` return types and should handle exceptions gracefully:

```csharp
client.OnError += async (exception) =>
{
    // Log the error
    logger.LogError(exception, "LNET client error occurred");
    
    // Implement recovery logic
    switch (exception)
    {
        case SocketException:
            // Network issue - may need reconnection
            break;
        case XmlException:
            // Parse error - library handles recovery automatically
            break;
        default:
            // Unknown error
            break;
    }
};
```

## Thread Safety

### Thread-Safe Operations
- All sending methods (`SendMessageAsync`, `JoinChannelAsync`, etc.)
- Property getters (`IsConnected`, `SupportedMessageTypes`, etc.)
- Statistics retrieval (`GetStatistics()`)
- Event subscription/unsubscription

### Single-Thread Operations
- Message receiving (`ReceiveAsync`, `GetMessageStreamAsync`)
- Event handler execution (called synchronously)

### Best Practices
- Use a single task/thread for message processing
- Avoid blocking operations in event handlers
- Use `ConfigureAwait(false)` in library code (already implemented)