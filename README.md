# LNET-Discord Bridge Application

## 🎯 Project Overview

This repository contains a complete application ecosystem that bridges LNET (LichNet) game servers with Discord, enabling real-time communication between in-game players and Discord users. The application demonstrates modern .NET 9 development practices, high-performance network programming, and containerized microservice architecture.

## 🧪 Educational Purpose & AI-Assisted Development

**This project serves as a learning experience and research initiative focused on:**

- **AI-Assisted Software Development**: Exploring best practices for leveraging Large Language Models (LLMs) in software engineering workflows
- **Cross-Language Translation**: Demonstrating the feasibility of translating existing applications from one programming language (Go) to another (C#) using AI assistance
- **Modern .NET Patterns**: Implementing current best practices in .NET 9, including performance optimizations, dependency injection, and containerization
- **Production-Ready Architecture**: Building resilient, scalable systems using industry-standard patterns and practices

### 🔄 Translation Project

This C# implementation is a translation and modernization of an existing GoLang application. The project explores:

- Language paradigm differences and adaptation strategies
- Performance characteristic preservation across languages
- Modern framework adoption during translation
- Architecture pattern evolution

## ⚠️ Disclaimer

**THIS SOFTWARE IS PROVIDED FOR EDUCATIONAL AND RESEARCH PURPOSES ONLY.**

- **No Liability**: The authors and contributors assume no liability for any damages, data loss, or issues arising from the use of this code
- **Educational Use**: This code is intended for learning, experimentation, and research purposes
- **Production Use**: Use in production environments is at your own risk and discretion
- **No Warranty**: This software is provided "as-is" without any warranties, express or implied
- **Learning Project**: This is an experimental project demonstrating AI-assisted development techniques

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   LNET Client   │◄───┤   Bridge Core   ├───►│ Discord Client  │
│    Library      │    │    Service      │    │    Library      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ GemStone IV/DR  │    │  Configuration  │    │    Discord      │
│   Game Servers  │    │   & Logging     │    │     Server      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📦 Components

### 🔌 LNet Library
High-performance .NET 9 client library for LNET protocol communication:

- **Span-based XML parsing** for zero-allocation performance
- **Production-grade resilience** with timeout handling, error recovery, and memory pressure management
- **SSL/TLS support** with certificate validation
- **Automatic keepalive** management
- **Channel-based message processing** using .NET 9 optimizations
- **Comprehensive logging** with ZLogger integration

**Key Features:**
- Buffer boundary protection for network data fragmentation
- Malformed XML recovery with circuit breaker patterns
- Adaptive memory management with multiple buffer pools
- Event-driven architecture with async/await patterns

### 🤖 Discord Library
Modern Discord bot client with advanced features:

- **Slash command support** with parameter validation
- **Embed-rich messaging** with dynamic content
- **Role-based permissions** and access control
- **Rate limiting** and API optimization
- **WebSocket connection management** with reconnection logic

### 🌉 Bridge Service
Central orchestration service that connects the two systems:

- **Message translation** between LNET and Discord protocols
- **User mapping** and authentication
- **Channel routing** and filtering
- **Command processing** and response handling
- **Health monitoring** and metrics collection

## 🚀 Getting Started

### Prerequisites

- **.NET 9.0 SDK** or later
- **Docker** (for containerized deployment)
- **LNET server access** (GemStone IV or DragonRealms account)
- **Discord Bot Token** and server permissions

### Quick Start

1. **Clone the repository:**
```bash
git clone <repository-url>
cd lnet-discord-bridge
```

2. **Configure the application:**
```bash
cp appsettings.example.json appsettings.json
# Edit appsettings.json with your configuration
```

3. **Run locally:**
```bash
dotnet restore
dotnet build
dotnet run --project src/BridgeService
```

4. **Run with Docker:**
```bash
docker build -t lnet-bridge .
docker run -d --name lnet-bridge \
  -v $(pwd)/config:/app/config \
  lnet-bridge
```

### Configuration Example

```json
{
  "LNet": {
    "Host": "lnet.lichproject.org",
    "User": "YourGameCharacter",
    "Game": "GSIV",
    "EnableKeepalive": true
  },
  "Discord": {
    "Token": "your-discord-bot-token",
    "GuildId": "your-discord-server-id",
    "ChannelMappings": {
      "ooc": "1234567890123456789",
      "general": "1234567890123456789"
    }
  }
}
```

## 🛠️ Development

### Project Structure

```
src/
├── LNet/                    # LNET client library
│   ├── Client.cs           # Main client implementation
│   ├── Messages.cs         # Message type definitions
│   ├── SpanXmlParser/      # High-performance XML parser
│   └── ...
├── Discord/                # Discord client library
│   ├── DiscordClient.cs    # Discord bot implementation
│   ├── Commands/           # Slash command handlers
│   └── ...
├── BridgeService/          # Main bridge application
│   ├── Program.cs          # Entry point and DI setup
│   ├── Services/           # Core business logic
│   └── ...
└── Shared/                 # Common utilities and models
```

### Building and Testing

```bash
# Restore dependencies
dotnet restore

# Build solution
dotnet build

# Run tests
dotnet test

# Run with development settings
dotnet run --project src/BridgeService --environment Development
```

### Docker Development

```bash
# Build development image
docker build -f Dockerfile.dev -t lnet-bridge:dev .

# Run with volume mounts for development
docker run -it --rm \
  -v $(pwd):/app \
  -v $(pwd)/config:/app/config \
  lnet-bridge:dev
```

## 📊 Performance Characteristics

### LNet Library Performance
- **Zero-allocation parsing** using ReadOnlySpan<byte>
- **~1000 messages/second** processing capability
- **Sub-millisecond latency** for message parsing
- **Adaptive memory usage** (16KB-1MB buffer sizes)
- **Production-tested resilience** with comprehensive error recovery

### Discord Library Performance
- **Rate limit compliance** with intelligent queuing
- **Batch message processing** for efficiency
- **Connection pooling** and persistent WebSocket connections
- **Graceful degradation** under API limits

## 🏭 Production Deployment

### Container Orchestration

**Kubernetes Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lnet-bridge
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: lnet-bridge
        image: lnet-bridge:latest
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
```

### Health Monitoring

The application exposes health check endpoints:
- `/health` - Overall application health
- `/health/lnet` - LNET connection status
- `/health/discord` - Discord connection status

### Logging and Metrics

- **Structured JSON logging** for log aggregation
- **Performance metrics** via built-in counters
- **Custom metrics** for business logic monitoring
- **Distributed tracing** support for troubleshooting

## 🔧 Configuration

### Environment Variables

```bash
# LNET Configuration
LNET__HOST=lnet.lichproject.org
LNET__USER=YourCharacter
LNET__GAME=GSIV

# Discord Configuration
DISCORD__TOKEN=your-bot-token
DISCORD__GUILDID=your-server-id

# Logging
LOGGING__LOGLEVEL__DEFAULT=Information
LOGGING__CONSOLE__ENABLED=true
```

### Advanced Configuration

- **SSL Certificate Validation**: Custom certificate validation callbacks
- **Parser Selection**: Choose between XmlReader and Span-based parsers
- **Buffer Management**: Configure buffer sizes and memory pressure handling
- **Retry Policies**: Exponential backoff and circuit breaker configuration

## 🤝 Contributing

This is an educational project exploring AI-assisted development. Contributions that further the learning objectives are welcome:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

### Code Style

- Follow **C# coding conventions**
- Use **async/await** patterns consistently
- Implement **proper error handling** and logging
- Write **comprehensive tests** for new features
- Document **public APIs** with XML comments

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Original GoLang Implementation**: This project builds upon concepts from an existing GoLang application
- **LNET Protocol**: Thanks to the Simutronics community for LNET documentation
- **Discord.NET Community**: For Discord integration patterns and best practices
- **AI-Assisted Development**: Exploring the frontier of human-AI collaboration in software engineering

## 📚 Learning Resources

### AI-Assisted Development
- Patterns for effective AI collaboration in coding
- Code review and validation techniques with LLMs
- Cross-language translation strategies and challenges

### .NET 9 Features Used
- **System.Threading.Channels** for high-performance message queuing
- **ArrayPool<T>** for memory-efficient buffer management
- **ReadOnlySpan<byte>** for zero-allocation parsing
- **IAsyncEnumerable** for streaming data processing
- **IHostedService** for long-running background services

### Performance Engineering
- Span-based parsing techniques
- Memory pressure monitoring and adaptive algorithms
- Network resilience patterns
- Container optimization strategies

---

**Remember**: This is an educational project. Use responsibly and at your own risk.
