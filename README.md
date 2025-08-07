# Claude Code Multi-Agent Observability & Communication System

A comprehensive platform for monitoring, visualizing, and enabling communication between Claude Code agents through advanced hook event tracking and inter-agent messaging. You can watch the [full breakdown here](https://youtu.be/9ijnN985O_c).

## 🎯 Overview

This dual-purpose system provides:

1. **Complete Observability**: Real-time monitoring and visualization of Claude Code agent behavior through [Hook events](https://docs.anthropic.com/en/docs/claude-code/hooks)
2. **Multi-Agent Communication**: Infrastructure for subagents to discover, message, and coordinate with each other

The platform captures, stores, and visualizes all agent activities while enabling sophisticated inter-agent collaboration patterns, making it ideal for complex multi-agent workflows. 

<img src="images/app.png" alt="Multi-Agent Observability Dashboard" style="max-width: 800px; width: 100%;">

## 🏗️ Architecture

```
Claude Agents → Hook Scripts → HTTP POST → Bun Server → SQLite → WebSocket → Vue Dashboard
                     ↓                          ↓
              Communication Hooks        Message Routing
                     ↓                          ↓
               Inter-Agent Messages      Subagent Registry
```

![Agent Data Flow Animation](images/AgentDataFlowV2.gif)

## 📋 Setup Requirements

Before getting started, ensure you have the following installed:

- **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** - Anthropic's official CLI for Claude
- **[Astral uv](https://docs.astral.sh/uv/)** - Fast Python package manager (required for hook scripts)
- **[Bun](https://bun.sh/)**, **npm**, or **yarn** - For running the server and client
- **Anthropic API Key** - Set as `ANTHROPIC_API_KEY` environment variable
- **OpenAI API Key** (optional) - For multi-model support with just-prompt MCP tool
- **ElevenLabs API Key** (optional) - For audio features

### Configure .claude Directory

To setup observability and multi-agent communication in your repo, copy the .claude directory to your project root.

To integrate the observability hooks into your projects:

1. **Copy the entire `.claude` directory to your project root:**
   ```bash
   cp -R .claude /path/to/your/project/
   ```

2. **Update the `settings.json` configuration:**
   
   Open `.claude/settings.json` in your project and modify the `source-app` parameter to identify your project:
   
   ```json
   {
     "hooks": {
       "PreToolUse": [{
         "matcher": "",
         "hooks": [
           {
             "type": "command",
             "command": "uv run .claude/hooks/pre_tool_use.py"
           },
           {
             "type": "command",
             "command": "uv run .claude/hooks/send_event.py --source-app YOUR_PROJECT_NAME --event-type PreToolUse --summarize"
           }
         ]
       }],
       "PostToolUse": [{
         "matcher": "",
         "hooks": [
           {
             "type": "command",
             "command": "uv run .claude/hooks/post_tool_use.py"
           },
           {
             "type": "command",
             "command": "uv run .claude/hooks/send_event.py --source-app YOUR_PROJECT_NAME --event-type PostToolUse --summarize"
           }
         ]
       }],
       "UserPromptSubmit": [{
         "hooks": [
           {
             "type": "command",
             "command": "uv run .claude/hooks/user_prompt_submit.py --log-only"
           },
           {
             "type": "command",
             "command": "uv run .claude/hooks/send_event.py --source-app YOUR_PROJECT_NAME --event-type UserPromptSubmit --summarize"
           }
         ]
       }]
       // ... (similar patterns for Notification, Stop, SubagentStop, PreCompact)
     }
   }
   ```
   
   Replace `YOUR_PROJECT_NAME` with a unique identifier for your project (e.g., `my-api-server`, `react-app`, etc.).

3. **Ensure the observability server is running:**
   ```bash
   # From the observability project directory (this codebase)
   ./scripts/start-system.sh
   ```

Now your project will send events to the observability system whenever Claude Code performs actions.

## 🚀 Quick Start

You can quickly view how this works by running this repositories .claude setup.

```bash
# 1. Start both server and client
./scripts/start-system.sh

# 2. Open http://localhost:5173 in your browser

# 3. Open Claude Code and run the following command:
Run git ls-files to understand the codebase.

# 4. Watch events stream in the client

# 5. Copy the .claude folder to other projects you want to emit events from.
cp -R .claude <directory of your codebase you want to emit events from>
```

## 📁 Project Structure

```
claude-code-hooks-multi-agent-observability/
│
├── apps/                    # Application components
│   ├── server/             # Bun TypeScript server
│   │   ├── src/
│   │   │   ├── index.ts    # Main server with HTTP/WebSocket endpoints
│   │   │   ├── db.ts       # SQLite database with 3 tables:
│   │   │   │               # - events (observability)
│   │   │   │               # - subagent_registry (agent tracking)
│   │   │   │               # - subagent_messages (inter-agent comms)
│   │   │   └── types.ts    # TypeScript interfaces
│   │   ├── package.json
│   │   └── events.db       # SQLite database (gitignored)
│   │
│   └── client/             # Vue 3 TypeScript dashboard
│       ├── src/
│       │   ├── App.vue     # Main app with dual tabs:
│       │   │               # - Event Timeline (observability)
│       │   │               # - Subagent Communications (messaging)
│       │   ├── components/
│       │   │   ├── EventTimeline.vue      # Event list with auto-scroll
│       │   │   ├── EventRow.vue           # Individual event display
│       │   │   ├── FilterPanel.vue        # Multi-select filters
│       │   │   ├── ChatTranscriptModal.vue # Chat history viewer
│       │   │   ├── SubagentComms.vue      # Message viewer & agent list
│       │   │   ├── StickScrollButton.vue  # Scroll control
│       │   │   └── LivePulseChart.vue     # Real-time activity chart
│       │   ├── composables/
│       │   │   ├── useWebSocket.ts        # WebSocket connection logic
│       │   │   ├── useEventColors.ts      # Color assignment system
│       │   │   ├── useChartData.ts        # Chart data aggregation
│       │   │   └── useEventEmojis.ts      # Event type emoji mapping
│       │   ├── utils/
│       │   │   └── chartRenderer.ts       # Canvas chart rendering
│       │   └── types.ts    # TypeScript interfaces
│       ├── .env.sample     # Environment configuration template
│       └── package.json
│
├── .claude/                # Claude Code integration
│   ├── hooks/             # Hook scripts (Python with uv)
│   │   ├── observability/ # Observability hooks
│   │   │   ├── send_event.py     # Universal event sender with AI summary
│   │   │   ├── pre_tool_use.py   # Tool validation & blocking
│   │   │   ├── post_tool_use.py  # Result logging
│   │   │   ├── notification.py   # User interaction events
│   │   │   ├── user_prompt_submit.py # User prompt logging
│   │   │   ├── stop.py          # Session completion with TTS
│   │   │   └── subagent_stop.py # Subagent completion with TTS
│   │   │
│   │   └── comms/         # Communication hooks
│   │       ├── send_message.py      # Send inter-agent messages
│   │       └── get_unread_messages.py # Retrieve unread messages
│   │
│   ├── agents/            # Agent configurations
│   │   └── hook-test-dummy.md  # Test agent for multi-agent comms
│   │
│   └── settings.json      # Hook configuration (currently commented out)
│
├── scripts/               # Utility scripts
│   ├── start-system.sh   # Launch server & client
│   ├── reset-system.sh   # Stop all processes
│   └── test-system.sh    # System validation
│
└── logs/                 # Application logs (gitignored)
```

## 🔧 Component Details

### 1. Hook System (`.claude/hooks/`)

> If you want to master claude code hooks watch [this video](https://github.com/disler/claude-code-hooks-mastery)

The hook system provides two distinct capabilities:

#### Observability Hooks (`.claude/hooks/observability/`)
Intercepts Claude Code lifecycle events:

- **`send_event.py`**: Core script that sends event data to the server
  - Supports `--add-chat` flag for including conversation history
  - AI-powered summarization via OpenAI/Anthropic APIs
  - Validates server connectivity before sending
  - Handles all event types with proper error handling

- **Event-specific hooks**: Each implements validation and data extraction
  - `pre_tool_use.py`: Blocks dangerous commands, validates tool usage
  - `post_tool_use.py`: Captures execution results and outputs
  - `notification.py`: Tracks user interaction points
  - `user_prompt_submit.py`: Logs user prompts, supports validation
  - `stop.py`: Records session completion with TTS notifications
  - `subagent_stop.py`: Monitors subagent completion with TTS alerts

#### Communication Hooks (`.claude/hooks/comms/`)
Enables inter-agent messaging:

- **`send_message.py`**: Broadcast messages to other subagents
  - Supports structured messages with type and data fields
  - Automatic JSON parsing for complex payloads
  - Session-independent messaging

- **`get_unread_messages.py`**: Retrieve unread messages
  - Filter by subagent name
  - Mark messages as read on retrieval
  - JSON or human-readable output formats

### 2. Server (`apps/server/`)

Bun-powered TypeScript server with comprehensive APIs:

- **Database**: SQLite with WAL mode and 3 tables:
  - `events`: Observability event storage
  - `subagent_registry`: Agent tracking and discovery
  - `subagent_messages`: Inter-agent message queue

- **Observability Endpoints**:
  - `POST /events` - Receive events from agents
  - `GET /events/recent` - Paginated event retrieval with filtering
  - `GET /events/filter-options` - Available filter values
  - `WS /stream` - Real-time event broadcasting

- **Communication Endpoints**:
  - `POST /subagents/register` - Register new subagents
  - `POST /subagents/message` - Send inter-agent messages
  - `POST /subagents/unread` - Get unread messages for agent
  - `GET /subagents/messages` - Retrieve all messages
  - `GET /subagents/:sessionId` - List agents in session

- **Features**:
  - Automatic schema migrations
  - Message read tracking
  - WebSocket broadcast to all clients
  - Chat transcript storage with AI summaries

### 3. Dashboard (`apps/client/`)

Vue 3 application with dual-purpose interface:

#### Event Timeline Tab
- **Visual Design**:
  - Dual-color system: App colors (left border) + Session colors (second border)
  - Gradient indicators for visual distinction
  - Dark/light theme support
  - Responsive layout with smooth animations

- **Features**:
  - Real-time WebSocket updates
  - Multi-criteria filtering (app, session, event type)
  - Live pulse chart with session-colored bars
  - Time range selection (1m, 3m, 5m)
  - Chat transcript viewer with syntax highlighting
  - Auto-scroll with manual override
  - Event limiting (configurable via `VITE_MAX_EVENTS_TO_DISPLAY`)

#### Subagent Communications Tab
- **Agent Registry**:
  - Live list of active subagents per session
  - Agent type and creation time tracking
  - Visual agent identification

- **Message Display**:
  - Real-time message updates
  - Sender identification with timestamps
  - Read receipt tracking
  - Message type categorization

- **Live Pulse Chart**:
  - Canvas-based real-time visualization
  - Session-specific colors for each bar
  - Event type emojis displayed on bars
  - Smooth animations and glow effects
  - Responsive to filter changes

## 🔄 Data Flows

### Observability Flow
1. **Event Generation**: Claude Code executes an action (tool use, notification, etc.)
2. **Hook Activation**: Corresponding hook script runs based on `settings.json` configuration
3. **Data Collection**: Hook script gathers context (tool name, inputs, outputs, session ID)
4. **AI Summarization**: Optional AI-powered summary generation
5. **Transmission**: `send_event.py` sends JSON payload to server via HTTP POST
6. **Server Processing**:
   - Validates event structure
   - Stores in SQLite with timestamp
   - Broadcasts to WebSocket clients
7. **Dashboard Update**: Vue app receives event and updates timeline in real-time

### Communication Flow
1. **Agent Registration**: Subagents auto-register when created via Task tool
2. **Message Sending**: Agent calls `send_message.py` with sender name and message
3. **Server Routing**: Message stored in `subagent_messages` table
4. **Message Retrieval**: Recipients poll using `get_unread_messages.py`
5. **Read Tracking**: Server marks messages as read for each recipient
6. **Dashboard Display**: Real-time updates show message exchange

## 🎨 Event Types & Visualization

| Event Type   | Emoji | Purpose               | Color Coding  | Special Display |
| ------------ | ----- | --------------------- | ------------- | --------------- |
| PreToolUse   | 🔧     | Before tool execution | Session-based | Tool name & details |
| PostToolUse  | ✅     | After tool completion | Session-based | Tool name & results |
| Notification | 🔔     | User interactions     | Session-based | Notification message |
| Stop         | 🛑     | Response completion   | Session-based | Summary & chat transcript |
| SubagentStop | 👥     | Subagent finished     | Session-based | Subagent details |
| PreCompact   | 📦     | Context compaction    | Session-based | Compaction details |
| UserPromptSubmit | 💬 | User prompt submission | Session-based | Prompt: _"user message"_ (italic) |


## 🧪 Testing

### System Testing
```bash
# System validation
./scripts/test-system.sh

# Manual event test
curl -X POST http://localhost:4000/events \
  -H "Content-Type: application/json" \
  -d '{
    "source_app": "test",
    "session_id": "test-123",
    "hook_event_type": "PreToolUse",
    "payload": {"tool_name": "Bash", "tool_input": {"command": "ls"}}
  }'
```

### Multi-Agent Communication Testing
```bash
# Send a message
python3 .claude/hooks/comms/send_message.py \
  --sender "Agent-Alpha" \
  --message "Hello from Alpha!"

# Check for messages
python3 .claude/hooks/comms/get_unread_messages.py \
  --name "Agent-Beta" \
  --json

# Test with hook-test-dummy agents
# In Claude Code, create two parallel test agents:
# Task 1: "Agent-Alpha: Test messaging system"
# Task 2: "Agent-Beta: Respond to messages"
```

## ⚙️ Configuration

### Environment Variables

Copy `.env.sample` to `.env` in the project root and fill in your API keys:

**Application Root** (`.env` file):
- `ANTHROPIC_API_KEY` – Anthropic Claude API key (required)
- `ENGINEER_NAME` – Your name (for logging/identification)
- `GEMINI_API_KEY` – Google Gemini API key (optional)
- `OPENAI_API_KEY` – OpenAI API key (optional)
- `ELEVEN_API_KEY` – ElevenLabs API key (optional)

**Client** (`.env` file in `apps/client/.env`):
- `VITE_MAX_EVENTS_TO_DISPLAY=100` – Maximum events to show (removes oldest when exceeded)

### Server Ports

- Server: `4000` (HTTP/WebSocket)
- Client: `5173` (Vite dev server)

## 📊 Technical Stack

- **Server**: Bun, TypeScript, SQLite (with WAL mode)
- **Dashboard**: Vue 3, TypeScript, Vite, Tailwind CSS
- **Hooks**: Python 3.8+, Astral uv
- **AI Features**: 
  - TTS: ElevenLabs, OpenAI, or pyttsx3
  - Summarization: OpenAI or Anthropic Claude
- **Communication**: HTTP REST, WebSocket, Inter-process messaging


Claude Code automatically sets `$CLAUDE_PROJECT_DIR` to your project root, ensuring hooks work regardless of the current working directory.

### Server Connection Issues

- Ensure the server is running: `./scripts/start-system.sh`
- Check server logs: `tail -f apps/server/server.log`
- Verify port 4000 is not in use: `lsof -i :4000`

### Message Delivery Problems

- Verify subagent names are unique
- Check server is running and accessible
- Review message logs in the dashboard's Subagent Communications tab

