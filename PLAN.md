# macOS Notes MCP Server Implementation Plan

## Overview
Build a Python MCP (Model Context Protocol) server to access the macOS Notes app database and enable intelligent querying through Claude Desktop.

## Project Structure
```
notes-querier-mcp/
├── PLAN.md                 # This implementation plan
├── requirements.txt        # Python dependencies
├── src/
│   ├── notes_reader.py    # Notes database access and parsing
│   ├── server.py          # MCP server implementation
│   └── __init__.py
├── tests/
│   └── test_notes_reader.py
└── config/
    └── claude_desktop_config.json  # Sample Claude Desktop config
```

## Technical Details

### AppleScript Notes Access
- **Approach**: Use AppleScript to access encrypted Notes data through the Notes.app API
- **Key Properties**:
  - `name` (text, read-only): The name/title of the note
  - `id` (text, read-only): Unique identifier for the note
  - `body` (text): HTML content of the note
  - `creation date` (date, read-only): When the note was created
  - `modification date` (date, read-only): When the note was last modified
  - `container` (account or folder, read-only): Where the note is stored
- **Data Format**: Note content is HTML markup, accessible without decryption
- **Accounts**: Supports iCloud, Exchange, and local accounts

### MCP Server Components

#### 1. AppleScript Notes Reader (`notes_reader.py`)
**Purpose**: Handle AppleScript execution and data parsing

**Key Functions**:
- Execute AppleScript commands to access Notes.app
- Parse HTML content from note bodies
- Extract note metadata (name, id, dates, container)
- Handle multiple account types (iCloud, Exchange, local)
- Process and clean HTML content for text extraction

#### 2. MCP Server Implementation (`server.py`)
**Purpose**: FastMCP server with STDIO transport

**MCP Tools to Implement**:
- `search_notes(query: str, limit: int = 10)`: Search notes by text content, title, or keywords
- `get_note(note_id: str)`: Retrieve specific note by ID or title
- `list_notes(folder: str = None, limit: int = 20)`: Get notes with metadata (title, date, folder)
- `get_note_attachments(note_id: str)`: Access embedded objects/media
- `get_folders()`: List all note folders/categories

#### 3. Dependencies & Setup
**Required Packages**:
- `fastmcp`: MCP server framework
- `subprocess`: Built-in Python for AppleScript execution
- `html2text` or `beautifulsoup4`: For HTML content parsing

**Setup Requirements**:
- Python 3.10+
- Virtual environment setup
- macOS system with Notes.app accessible
- AppleScript permissions for automation
- Logging configuration (stderr only, never stdout for STDIO servers)

## Implementation Steps

### Phase 1: AppleScript Testing
1. Create basic AppleScript commands to access Notes.app
2. Test reading note properties and content
3. Explore account and folder enumeration
4. Document AppleScript command patterns

### Phase 2: Notes Reader Development
1. Implement AppleScript execution wrapper in Python
2. Build HTML content parsing and text extraction
3. Extract note metadata from AppleScript results
4. Handle different account types and folder structures
5. Process and clean HTML content for search functionality

### Phase 3: MCP Server Implementation
1. Set up FastMCP server with STDIO transport
2. Implement core tools (search_notes, get_note, list_notes)
3. Add proper type hints and docstrings
4. Implement error handling and logging
5. Test server functionality locally

### Phase 4: Claude Desktop Integration
1. Create Claude Desktop configuration
2. Test MCP server connection
3. Validate tool functionality through Claude Desktop
4. Configure appropriate AppleScript automation permissions

### Phase 5: Advanced Features
1. Implement semantic search capabilities
2. Add date-based filtering
3. Support for note attachments/media
4. Performance optimization for large note collections

## Usage Examples

Once implemented, you'll be able to query your notes through Claude Desktop:

- "Find notes about project planning from this month"
- "Show me all notes containing 'meeting' in the title"
- "Get the note I wrote about the quarterly review"
- "List all notes in my Work folder"
- "Find notes with attachments or images"

## Configuration

### Claude Desktop Config
```json
{
  "mcpServers": {
    "notes": {
      "command": "python",
      "args": ["/path/to/notes-querier-mcp/src/server.py"],
      "cwd": "/path/to/notes-querier-mcp"
    }
  }
}
```

## Security Considerations
- Notes data contains personal information
- AppleScript provides read-only access by default
- Handle AppleScript permission requests gracefully
- Respect macOS privacy and security permissions
- Process HTML content safely to prevent injection attacks

## Testing Strategy
- Unit tests for AppleScript execution and parsing
- Mock AppleScript responses for testing without Notes.app
- Integration tests with actual Notes data
- Validation of MCP protocol compliance
- HTML content parsing accuracy tests

## Future Enhancements
- Support for note creation/modification (if needed)
- Full-text search indexing for better performance
- Export functionality to various formats
- Backup and sync capabilities
- Web interface for note management