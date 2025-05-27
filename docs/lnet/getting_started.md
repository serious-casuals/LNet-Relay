---
layout: default
title: Getting Started
parent: LNet Library
nav_order: 1
---

# Getting Started with LNet Library

This guide will walk you through setting up and using the LNet library to connect to LNET servers for GemStone IV and DragonRealms.

## Installation

### Package Manager

```bash
dotnet add package LNet
```

### PackageReference

```xml
<PackageReference Include="LNet" Version="2.0.0" />
```

## Prerequisites

Before you begin, you'll need:

1. **.NET 9.0 SDK** or later
2. **LNET server access** - A valid GemStone IV or DragonRealms account
3. **Network connectivity** to `lnet.lichproject.org` on port 7155

## Your First Connection

### Basic Connection

```csharp
using LNet;
using Microsoft.Extensions.Logging;

// Create a simple console logger
using var loggerFactory = LoggerFactory.Create(builder =>
    builder.AddConsole().SetMinimumLevel(LogLevel.Information));

// Configure connection options
var options = new Options
{
    Host = "lnet.lichproject.org",
    User = "YourCharacterName",
    Game = "GSIV", // Use "DR" for DragonRealms
    Debug = true,
    EnableKeepalive = true,
    AcceptAnyCertificate = false // Set to true only for testing
};

try
{
    // Create and connect
    using var client = await Client.CreateAsync(options, loggerFactory);
    
    Console.WriteLine($"Connected! Uptime: {client.Uptime}");
    Console.WriteLine($"Supported message types: {string.Join(", ", client.SupportedMessageTypes)}");
    
    // Receive a single message
    var message = await client.ReceiveAsync();
    Console.WriteLine($"First message: {message}");
}
catch (Exception ex)
{
    Console.WriteLine($"Connection failed: {ex.Message}");
}
```

### Message Processing Loop

```csharp
using var client = await Client.CreateAsync(options, loggerFactory);

// Process messages continuously
await foreach (var message in client.GetMessageStreamAsync())
{
    Console.WriteLine($"Received: {message.MessageType} at {message.ReceivedAt}");
    
    // Handle different message types
    switch (message)
    {
        case Message msg:
            Console.WriteLine($"Message from {msg.From}: {msg.Text}");
            break;
            
        case Ping:
            Console.WriteLine("Received ping (auto-handled)");
            break;
            
        case LocationRequest req:
            Console.WriteLine($"Location request from {req.User}");
            break;
            
        default:
            Console.WriteLine($"Unknown message type: {message.GetType().Name}");
            break;
    }
}
```

## Event-Driven Approach

For more responsive applications, use the event-driven pattern:

```csharp
var client = await Client.CreateAsync(options, loggerFactory);

// Set up event handlers
client.OnConnected += async () =>
{
    Console.WriteLine("🔗 Connected to LNET!");
    
    // Join channels after connection
    await client.JoinChannelAsync("ooc");
    await client.JoinChannelAsync("general");
    
    // Set your status
    await client.SendStatusAsync("online", "Connected via LNet library");
};

client.OnDisconnected += async () =>
{
    Console.WriteLine("❌ Disconnected from LNET");
};

client.OnError += async (exception) =>
{
    Console.WriteLine($"💥 Error: {exception.Message}");
};

client.OnMessageReceived += async (message) =>
{
    // This fires for EVERY message before it's queued
    Console.WriteLine($"📨 Incoming: {message.MessageType}");
};

// Keep the application running
Console.WriteLine("Press Ctrl+C to exit...");
Console.CancelKeyPress += (s, e) => {
    e.Cancel = true;
    client.Dispose();
};

// Process messages in background
_ = Task.Run(async () =>
{
    await foreach (var message in client.GetMessageStreamAsync())
    {
        await ProcessMessage(message);
    }
});

// Keep main thread alive
await Task.Delay(Timeout.Infinite);
```

## Sending Messages

### Join/Leave Channels

```csharp
// Join channels
await client.JoinChannelAsync("ooc");
await client.JoinChannelAsync("general");

// Leave channels
await client.LeaveChannelAsync("ooc");
```

### Send Messages

```csharp
// Private message
var privateMsg = new Message(
    type: "private",
    to: "SomePlayer",
    text: "Hello from the LNet library!"
);
await client.SendMessageAsync(privateMsg);

// Channel message (if supported by server)
var channelMsg = new Message(
    type: "channel",
    channel: "ooc",
    text: "Hello everyone!"
);
await client.SendMessageAsync(channelMsg);
```

### Other Operations

```csharp
// Manual ping (keepalive is automatic)
await client.SendPingAsync();

// Location request
await client.SendLocationRequestAsync("SomePlayer");

// Status update
await client.SendStatusAsync("away", "Be right back");
```

## Configuration Options

### Basic Options

```csharp
var options = new Options
{
    // Connection settings
    Host = "lnet.lichproject.org",
    User = "YourCharacter",
    Game = "GSIV", // or "DR"
    
    // Timeouts
    ConnectionTimeoutSeconds = 30,
    
    // Keepalive
    EnableKeepalive = true,
    KeepaliveIntervalSeconds = 60,
    
    // Debugging
    Debug = false,
    EnableFileLogging = false
};
```

### Advanced Options

```csharp
var options = new Options
{
    // ... basic settings ...
    
    // Performance
    MaxMessageQueueSize = 2000,
    ParserType = XmlParserType.SpanBased, // or Auto, XmlReader
    
    // Security
    AcceptAnyCertificate = false,
    CertificateValidationCallback = (sender, cert, chain, errors) => 
    {
        // Custom certificate validation logic
        return errors == SslPolicyErrors.None;
    },
    
    // Logging
    LogFilePath = "/app/logs/lnet.log"
};
```

## Error Handling

### Connection Errors

```csharp
try
{
    var client = await Client.CreateAsync(options);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Configuration error: {ex.Message}");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Connection error: {ex.Message}");
}
catch (TimeoutException ex)
{
    Console.WriteLine($"Connection timeout: {ex.Message}");
}
```

### Runtime Errors

```csharp
client.OnError += async (exception) =>
{
    switch (exception)
    {
        case SocketException socketEx:
            Console.WriteLine($"Network error: {socketEx.Message}");
            // Implement reconnection logic
            break;
            
        case XmlException xmlEx:
            Console.WriteLine($"Parse error: {xmlEx.Message}");
            // Parser will automatically recover
            break;
            
        default:
            Console.WriteLine($"Unexpected error: {exception}");
            break;
    }
};
```

## Best Practices

### 1. Use Proper Disposal

```csharp
// Always use 'using' statements
using var client = await Client.CreateAsync(options);

// Or dispose manually
var client = await Client.CreateAsync(options);
try
{
    // Use client
}
finally
{
    await client.DisposeAsync(); // Async disposal preferred
}
```

### 2. Handle Disconnections

```csharp
client.OnDisconnected += async () =>
{
    Console.WriteLine("Connection lost, implementing backoff...");
    
    // Implement exponential backoff
    var delay = TimeSpan.FromSeconds(5);
    for (int attempt = 1; attempt <= 3; attempt++)
    {
        try
        {
            await Task.Delay(delay);
            client = await Client.CreateAsync(options);
            Console.WriteLine("Reconnected successfully!");
            break;
        }
        catch
        {
            delay = TimeSpan.FromSeconds(delay.TotalSeconds * 2);
            Console.WriteLine($"Reconnection attempt {attempt} failed");
        }
    }
};
```

### 3. Monitor Performance

```csharp
// Check statistics periodically
var timer = new PeriodicTimer(TimeSpan.FromMinutes(5));
while (await timer.WaitForNextTickAsync())
{
    var stats = client.GetStatistics();
    Console.WriteLine($"Stats: {stats}");
    
    // Check for issues
    if (stats.ParserStatistics is SpanParserStatistics spanStats)
    {
        if (spanStats.ParseErrors > 100)
        {
            Console.WriteLine("High parse error rate detected!");
        }
    }
}
```

### 4. Graceful Shutdown

```csharp
// Handle Ctrl+C gracefully
var cts = new CancellationTokenSource();
Console.CancelKeyPress += (s, e) => {
    e.Cancel = true;
    cts.Cancel();
};

try
{
    await foreach (var message in client.GetMessageStreamAsync(cts.Token))
    {
        await ProcessMessage(message);
    }
}
catch (OperationCanceledException)
{
    Console.WriteLine("Shutting down gracefully...");
}
```

## Next Steps

Now that you have a basic connection working:

1. **Explore [Message Types]({% link lnet/api.md %}#message-types)** to understand all available message structures
2. **Check out [Examples]({% link lnet/examples.md %})** for more complex scenarios
3. **Learn about [Performance]({% link lnet/performance.md %})** optimization techniques
4. **Review [Configuration]({% link lnet/configuration.md %})** for production deployment

## Troubleshooting

### Common Issues

**Connection Timeout:**
```
InvalidOperationException: Failed to connect to lnet.lichproject.org:7155
```
- Check network connectivity
- Verify LNET server is running
- Try increasing `ConnectionTimeoutSeconds`

**SSL Certificate Errors:**
```
AuthenticationException: The remote certificate is invalid
```
- Set `AcceptAnyCertificate = true` for testing only
- Implement custom `CertificateValidationCallback`

**Parse Errors:**
```
XmlException: Data at the root level is invalid
```
- Check network stability
- Monitor parser statistics for error patterns
- Parser automatically recovers from malformed XML

**Memory Issues:**
```
OutOfMemoryException during buffer operations
```
- Reduce `MaxMessageQueueSize`
- Monitor GC pressure
- Check for message processing bottlenecks