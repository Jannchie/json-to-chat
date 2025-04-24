# JSON to Chat Converter

A Vue.js application that converts JSON chat data into a visual chat interface. This tool allows you to:

- Edit JSON chat data in a Monaco-based editor
- Preview the chat in real-time
- Share conversations via URL with base64-encoded JSON
- Validate JSON structure with detailed error messages

## Features

- Split-pane interface with JSON editor and chat preview
- Real-time validation and error reporting
- Support for system, user, and assistant message roles
- Shareable URLs for easy distribution of chat conversations

## Usage

### Install

```bash
pnpm install
```

### Development

```bash
pnpm dev
```

### Build

```bash
pnpm build
```

### Lint

```bash
pnpm lint
```

### Test

```bash
pnpm test
```

## JSON Structure

The application expects JSON in the following format:

```json
[
  {
    "role": "system",
    "content": "You are a helpful AI assistant."
  },
  {
    "role": "user",
    "content": "Hello, can you help me with a coding problem?"
  },
  {
    "role": "assistant",
    "content": "Of course! I'd be happy to help with your coding problem. What specifically are you working on?"
  }
]
```

Supported roles:

- `system`: System instructions or context
- `user`: User messages
- `assistant`: AI assistant responses
