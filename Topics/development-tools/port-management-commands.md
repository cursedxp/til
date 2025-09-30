# Port Management Commands for Development

## Overview

Essential commands for managing ports during local development, troubleshooting "port already in use" errors, and managing multiple Node.js processes efficiently.

## Check What's Running on Specific Ports

### Single Port
```bash
# Check what's using port 3000
lsof -i :3000
```

### Multiple Ports
```bash
# Check multiple ports at once
lsof -i :3000,3001,8080
```

### Filter by Status
```bash
# Only show listening processes
lsof -i :3000 | grep LISTEN

# Show all connections
lsof -i :3000
```

### Detailed Information
```bash
# Get detailed process info with port numbers
lsof -i :3000 -P
```

## Kill Processes on Ports

### Kill All Processes on a Port
```bash
# Standard kill
lsof -ti:3000 | xargs kill

# Force kill if process won't stop
lsof -ti:3000 | xargs kill -9
```

### Kill Specific Process by PID
```bash
# First, find the PID
lsof -i :3000

# Then kill it
kill [PID_NUMBER]

# Force kill if needed
kill -9 [PID_NUMBER]
```

### Verify Port is Free
```bash
# Check if port is now available
lsof -i :3000

# Should return empty if port is free
```

## Understanding Port Output

### Common Terms

- **hbci** = Port 3000 in system notation
- **LISTEN** = Server accepting connections (your dev server)
- **ESTABLISHED** = Active connection (like browser connected to server)

### Example Output
```
COMMAND   PID  USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
node     1234  user   23u  IPv4  0x1234567      0t0  TCP *:hbci (LISTEN)
chrome   5678  user   45u  IPv4  0x7654321      0t0  TCP localhost:hbci->localhost:12345 (ESTABLISHED)
```

**What this means:**
- **Line 1**: Node.js server listening on port 3000
- **Line 2**: Chrome browser connected to that server

## Common Scenarios

### Scenario 1: Port Already in Use
```bash
# Problem: "Port 3000 is already in use"

# Step 1: Find what's using it
lsof -i :3000

# Step 2: Kill the process
lsof -ti:3000 | xargs kill

# Step 3: Verify it's free
lsof -i :3000

# Step 4: Start your server
npm run dev
```

### Scenario 2: Zombie Processes
```bash
# Problem: Killed terminal but process still running

# Step 1: Find the zombie process
lsof -i :3000

# Step 2: Force kill
lsof -ti:3000 | xargs kill -9

# Step 3: Confirm it's gone
lsof -i :3000
```

### Scenario 3: Multiple Chrome Connections
```
# Multiple Chrome entries are NORMAL
# Hot reload creates multiple connections
# Only kill if:
# - Closing browser doesn't free the port
# - Process is from old session
```

## Development Workflow

### When Switching Projects
```bash
# 1. Stop current dev server (Ctrl+C)
# 2. Verify port is free
lsof -i :3000
# 3. Kill if needed
lsof -ti:3000 | xargs kill
# 4. Start new project
npm run dev
```

### Clean Slate Development Start
```bash
# Kill all common dev ports
lsof -ti:3000,3001,8080 | xargs kill

# Verify all are free
lsof -i :3000,3001,8080

# Start fresh
npm run dev
```

## Platform-Specific Notes

### macOS
All commands above work on macOS (Darwin kernel)

### Linux
Same commands, `lsof` usually pre-installed

### Windows
Use alternative commands:
```powershell
# Find process on port
netstat -ano | findstr :3000

# Kill process by PID
taskkill /PID [PID_NUMBER] /F
```

## Best Practices

1. **Always verify**: Check port is free after killing processes
2. **Graceful shutdown**: Try normal kill before force kill (`-9`)
3. **Know your ports**: Document which projects use which ports
4. **Clean up regularly**: Kill unused dev servers before switching projects
5. **Use different ports**: Avoid conflicts by assigning unique ports to projects

## Troubleshooting Tips

### Port Shows Free But Still Errors
```bash
# Wait a few seconds
sleep 2

# Check again
lsof -i :3000

# Try different port temporarily
PORT=3001 npm run dev
```

### Permission Denied
```bash
# Some ports (< 1024) require sudo
sudo lsof -i :80

# For development, use ports > 1024
# 3000, 3001, 8080, etc.
```

### Process Won't Die
```bash
# Ultimate force kill
sudo kill -9 [PID]

# Check if it's a system process
ps aux | grep [PID]
```

## Related Topics
- [NPM Dependencies vs VS Code Extensions](./npm-dependencies-vs-vscode-extensions.md)
- [Next.js Development Workflow](../nextjs/nextjs-ssr-data-fetching-best-practices.md)