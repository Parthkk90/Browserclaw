# ClawLess: Browser-Based AI Agent Container

<div align="center">

![ClawLess Logo](./public/logo.png)

**A serverless, browser-based runtime for running AI agents directly in your browser**

Built with WebContainers • Powered by TypeScript • Secured by Policy Engine

[Documentation](./DOCS.md) • [Features](#features) • [Quick Start](#quick-start) • [API Reference](#api-reference)

</div>

---

## 🎯 What is ClawLess?

**ClawLess** is a powerful browser-based container runtime that allows AI agents to execute code, manage files, and interact with git repositories—all directly in your web browser. No servers required. No deployments needed. Just pure browser power.

Think of it as a lightweight, sandboxed development environment that runs entirely in the browser, equipped with:
- **Built-in Terminal** - Full Linux-like shell environment
- **Code Editor** - Monaco Editor with syntax highlighting
- **Git Integration** - Clone, commit, and push repositories
- **AI Agent Support** - Designed for autonomous agent frameworks like GitAgent
- **Security Policies** - Guardrails and audit logging for safe execution
- **Plugin System** - Extend functionality with custom plugins

### Perfect For:
- 🤖 Running AI agents in your browser
- 📝 Code generation and editing
- 🔄 Git-based workflows without backend servers
- 🧪 Interactive coding environments
- 🛡️ Sandboxed code execution with audit trails
- 🔌 Custom integrations via plugins

---

## ✨ Features

### 🌐 Browser-Native Runtime
- **WebContainer-Powered**: Full Node.js environment in your browser
- **No Server Needed**: Everything runs client-side
- **Instant Startup**: Boot a complete environment in seconds
- **Offline Capable**: Works without internet after initial load

### 💻 Developer Tools
- **Interactive Terminal**: Full xterm.js integration for shell commands
- **Code Editor**: Monaco Editor with language support
- **File System Access**: Read, write, and manage files
- **Real-time Output**: See results as they happen

### 🤖 AI Agent Integration
- **GitAgent Compatible**: Built on GitAgent standards
- **Command Execution**: Run arbitrary shell commands
- **Workspace Templates**: Pre-configured environments
- **Startup Scripts**: Custom initialization per project

### 🔐 Security & Compliance
- **Policy Engine**: Define execution guardrails (what can/cannot run)
- **Audit Logging**: Complete activity logs for compliance
- **Network Interception**: Monitor and control HTTP requests
- **Whitelisting**: Fine-grained control over operations

### 🔌 Extensibility
- **Plugin Architecture**: Add custom functionality
- **Event System**: React to container lifecycle events
- **Custom Tabs**: Define custom UI components
- **Template System**: Reusable container configurations

### 📦 Git Integration
- **Clone Repositories**: Pull code directly into container
- **Commit & Push**: Write changes back to git
- **Token Support**: GitHub token authentication
- **Automatic Sync**: Optional auto-sync on file changes

---

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/clawless.git
cd clawless

# Install dependencies
npm install

# Start development server
npm run dev
```

The app will be available at `http://localhost:5173` (Vite default).

### Basic Usage

#### 1. **HTML Setup**
```html
<!DOCTYPE html>
<html>
  <head>
    <title>ClawLess Demo</title>
    <link rel="stylesheet" href="dist/style.css" />
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="dist/sdk.js"></script>
  </body>
</html>
```

#### 2. **Initialize Container**
```javascript
import { ClawContainer } from './dist/sdk.js';

// Create and start container
const cc = new ClawContainer('#app', {
  workspace: {
    'hello.js': 'console.log("Hello, World!");'
  }
});

await cc.start();
```

#### 3. **Execute Commands**
```javascript
// Run shell commands
const output = await cc.exec('node hello.js');
console.log(output); // "Hello, World!"

// Use file system
await cc.fs.write('package.json', '{"name": "myapp"}');
const content = await cc.fs.read('package.json');
```

#### 4. **Clone from Git**
```javascript
// Clone a repository
await cc.git.clone('https://github.com/owner/repo', 'YOUR_GITHUB_TOKEN');

// Make changes and push back
await cc.fs.write('src/main.js', 'console.log("Updated!");');
await cc.git.push('Update main.js');
```

---

## 📚 API Reference

### ClawContainer Class

#### Constructor
```typescript
new ClawContainer(selector: string, options?: ClawContainerOptions)
```

**Parameters:**
- `selector` (string) - CSS selector for DOM element to mount to
- `options` (optional) - Configuration object

#### Lifecycle Methods

```typescript
// Boot the container and all services
await cc.start(): Promise<void>

// Gracefully shutdown
await cc.stop(): Promise<void>

// Restart the container
await cc.restart(): Promise<void>
```

#### Command Execution

```typescript
// Execute a shell command and get output
await cc.exec(command: string): Promise<string>

// Open an interactive shell
await cc.shell(): Promise<void>

// Send input to running process
await cc.sendInput(data: string): Promise<void>
```

#### File System (`cc.fs`)

```typescript
// Read file contents
await cc.fs.read(path: string): Promise<string>

// Write contents to file
await cc.fs.write(path: string, content: string): Promise<void>

// List directory contents
await cc.fs.list(dir?: string): Promise<string[]>

// Create directory
await cc.fs.mkdir(path: string): Promise<void>

// Delete file or directory
await cc.fs.remove(path: string): Promise<void>
```

#### Git Service (`cc.git`)

```typescript
// Clone a repository
await cc.git.clone(url: string, token?: string): Promise<void>

// Commit and push changes
await cc.git.push(message?: string): Promise<string>
```

#### Events

```typescript
// Container is ready
cc.on('ready', () => {});

// Error occurred
cc.on('error', (error: Error) => {});

// Status changed
cc.on('status', (status: string) => {});

// File changed
cc.on('file.change', (path: string) => {});

// Audit entry created
cc.on('log', (entry: AuditEntry) => {});
```

### Configuration Options

```typescript
interface ClawContainerOptions {
  // AI Agent configuration
  agent?: AgentConfig | false;
  
  // Initial workspace files
  workspace?: Record<string, string>;
  
  // Environment variables
  env?: Record<string, string>;
  
  // Startup shell script
  startupScript?: string;
  
  // Container template (pre-configured environment)
  template?: string | ContainerTemplate;
  
  // Custom plugins
  plugins?: ClawContainerPlugin[];
  
  // Custom UI tabs
  tabs?: TabDefinition[];
}
```

---

## 🔒 Security & Policies

### Define Execution Policies

Create YAML-based policies to control what code can execute:

```yaml
policies:
  - name: "Sandbox Mode"
    enforceRules:
      - glob: "*.js"
        allow: true
      - glob: "rm -rf /"
        allow: false
        action: "block"
      - glob: "http://internal.example.com/*"
        allow: false
        action: "log_and_block"
```

### Audit Logging

Every action is logged:

```javascript
// Access audit log
cc.audit.entries.forEach(entry => {
  console.log(entry.event, entry.timestamp, entry.details);
});

// Subscribe to new entries
cc.on('log', (entry) => {
  console.log('Action:', entry.event);
});
```

---

## 🔌 Plugins

Extend ClawLess with custom functionality:

```typescript
const myPlugin: ClawContainerPlugin = {
  name: 'MyPlugin',
  version: '1.0.0',
  
  onReady(cc) {
    console.log('Container is ready!');
    // Add custom UI, listen to events, etc.
  },
  
  onDestroy(cc) {
    console.log('Container shutting down');
  }
};

const cc = new ClawContainer('#app', {
  plugins: [myPlugin]
});
```

---

## 📋 Configuration

### Environment Variables

Set via constructor options:

```javascript
const cc = new ClawContainer('#app', {
  env: {
    'NODE_ENV': 'production',
    'API_KEY': 'your-key-here'
  }
});
```

### Templates

Pre-configure entire environments:

```javascript
const cc = new ClawContainer('#app', {
  template: 'nextjs' // or provide custom template object
});
```

### Agent Support

Configure AI agents:

```javascript
const cc = new ClawContainer('#app', {
  agent: {
    name: 'MyAgent',
    model: 'gpt-4',
    provider: 'openai'
  }
});
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│         ClawContainer (SDK)                 │
├─────────────────────────────────────────────┤
│  UIManager       PluginManager    TabManager│
│  TerminalMgr     ContainerMgr     GitService│
│  PolicyEngine    AuditLog         Events    │
├─────────────────────────────────────────────┤
│         WebContainer (Browser API)          │
├─────────────────────────────────────────────┤
│    Node.js Runtime + File System            │
└─────────────────────────────────────────────┘
```

### Core Modules

- **`sdk.ts`** - Main SDK class and public API
- **`container.ts`** - WebContainer management
- **`terminal.ts`** - Terminal emulation (xterm)
- **`ui.ts`** - UI components and layout
- **`plugin.ts`** - Plugin system
- **`policy.ts`** - Policy engine and guardrails
- **`audit.ts`** - Audit logging
- **`git-service.ts`** - Git operations
- **`net-intercept.ts`** - Network request interception
- **`monaco-editor.ts`** - Code editor integration
- **`templates.ts`** - Template registry and management
- **`types.ts`** - TypeScript type definitions
- **`event-emitter.ts`** - Event system
- **`workspace.ts`** - Workspace utilities

---

## 🛠️ Development

### Build

```bash
# Development build with hot reload
npm run dev

# Production build
npm run build

# Library build (for npm package)
npm run build:lib

# Preview production build
npm run preview
```

### Project Structure

```
clawless-main/
├── src/
│   ├── main.ts              # Entry point
│   ├── sdk.ts               # Main SDK class
│   ├── container.ts         # Container management
│   ├── terminal.ts          # Terminal emulation
│   ├── ui.ts                # UI management
│   ├── plugin.ts            # Plugin system
│   ├── policy.ts            # Policy engine
│   ├── audit.ts             # Audit logging
│   ├── git-service.ts       # Git integration
│   ├── net-intercept.ts     # Network interception
│   ├── monaco-editor.ts     # Code editor
│   ├── types.ts             # Type definitions
│   ├── event-emitter.ts     # Event system
│   ├── templates.ts         # Templates
│   ├── workspace.ts         # Workspace utils
│   ├── style.css            # Styles
│   └── ...                  # Other modules
├── public/                  # Static assets
├── index.html               # Entry HTML
├── package.json
├── tsconfig.json
├── vite.config.ts
└── DOCS.md                  # Technical docs
```

### Dependencies

- **@webcontainer/api** - Core WebContainer runtime
- **@xterm/xterm** - Terminal emulation
- **monaco-editor** - Code editor
- **jszip** - Archive handling
- **typescript** - Language support
- **vite** - Build tooling

---

## 📖 Examples

### Example 1: Simple Node.js Project

```javascript
const cc = new ClawContainer('#app', {
  workspace: {
    'package.json': JSON.stringify({ name: 'demo', version: '1.0.0' }),
    'index.js': 'console.log("Hello from ClawLess!");'
  }
});

await cc.start();
await cc.exec('npm install');
await cc.exec('node index.js');
```

### Example 2: Clone and Modify a Repository

```javascript
const cc = new ClawContainer('#app');
await cc.start();

await cc.git.clone('https://github.com/owner/repo', token);
const files = await cc.fs.list('.');
console.log('Cloned files:', files);

await cc.fs.write('README.md', '# Updated Readme');
await cc.git.push('Update documentation');
```

### Example 3: Run an AI Agent

```javascript
const cc = new ClawContainer('#app', {
  agent: {
    name: 'CodeAssistant',
    model: 'gpt-4',
    provider: 'openai'
  },
  plugins: [myCustomPlugin]
});

await cc.start();

// Agent can now execute commands, modify files, and interact with git
cc.on('status', status => console.log('Agent:', status));
```

### Example 4: Custom Policy

```javascript
const policyYaml = `
rules:
  - glob: "*.py"
    allow: true
  - glob: "rm -rf /"
    allow: false
    action: block
  - glob: "curl http://malicious.site"
    allow: false
    action: log_and_block
`;

const cc = new ClawContainer('#app');
await cc.start();

const policy = ClawContainer.parseTemplate(policyYaml);
cc.policy.loadPolicy(policy);
```

---

## 🐛 Troubleshooting

### Container Won't Boot
- Check browser console for errors
- Ensure your browser supports WebContainers (Chrome, Edge, Firefox)
- Try clearing localStorage: `localStorage.clear()`

### File Operations Failing
- Verify file path is correct
- Check disk quota hasn't been exceeded
- Try restarting the container

### Git Clone Issues
- Verify URL format: `https://github.com/owner/repo`
- Ensure token is valid if using private repos
- Check network connectivity

### Performance Issues
- Close unused tabs and terminals
- Limit file sizes in workspace
- Check browser memory usage in DevTools

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing`)
5. Open a Pull Request

---

## 📄 License

MIT License - see LICENSE file for details

---

## 🙏 Acknowledgments

- Built by **Shreyas Kapale** @ **Lyzr**
- Powered by **WebContainers**
- Inspired by **GitAgent** standard
- UI components from **Monaco Editor** and **xterm.js**

---

## 📞 Support & Resources

- **GitHub Issues**: [Report bugs or request features](https://github.com/yourusername/clawless/issues)
- **Documentation**: See [DOCS.md](./DOCS.md) for technical details
- **Examples**: Check the [examples](./examples) folder
- **Community**: Join discussions in GitHub Discussions

---

## 🗺️ Roadmap

- [ ] WebAssembly support for compiled languages
- [ ] Docker image export
- [ ] Persistent storage backend
- [ ] Multi-container orchestration
- [ ] Advanced debugging tools
- [ ] Browser extensions
- [ ] Cloud sync capabilities

---

<div align="center">

**Made with ❤️ for developers by developers**

[⭐ Star us on GitHub](https://github.com/yourusername/clawless) • [Report Issues](https://github.com/yourusername/clawless/issues) • [Documentation](./DOCS.md)

</div>
