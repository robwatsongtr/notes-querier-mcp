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

### macOS Notes Database
- **Location**: `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite`
- **Key Tables**:
  - `ZICCLOUDSYNCINGOBJECT`: Contains note metadata, titles, folder info
  - `ZICNOTEDATA`: Contains actual note content in compressed protobuf format
- **Data Format**: Note content is gzipped protobuf data in `ZICNOTEDATA.ZDATA` column
- **Schema Evolution**: Database structure changed significantly after iOS 9

### MCP Server Components

#### 1. Notes Database Reader (`notes_reader.py`)
**Purpose**: Handle SQLite database access and data parsing

**Key Functions**:
- Connect to Notes SQLite database
- Parse gzipped protobuf data from ZICNOTEDATA table
- Extract note metadata from ZICCLOUDSYNCINGOBJECT table
- Handle both local and iCloud-synced notes
- Parse embedded objects/attachments

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
- `protobuf`: For parsing protobuf data
- `sqlite3`: Built-in Python SQLite support

**Setup Requirements**:
- Python 3.10+
- Virtual environment setup
- Error handling for database permissions
- Logging configuration (stderr only, never stdout for STDIO servers)

## Implementation Steps

### Phase 1: Database Exploration
1. Create basic SQLite connection to Notes database
2. Explore schema and understand table relationships
3. Test data extraction from key tables
4. Document database structure findings

### Phase 2: Notes Reader Development
1. Implement SQLite connection with error handling
2. Build protobuf data decompression (gzip)
3. Parse note content from ZICNOTEDATA.ZDATA
4. Extract metadata from ZICCLOUDSYNCINGOBJECT
5. Handle different note types and embedded objects

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
4. Configure appropriate database access permissions

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
- Notes database contains personal information
- Implement read-only access to prevent accidental modifications
- Handle encrypted/password-protected notes gracefully
- Respect macOS privacy and security permissions

## Testing Strategy
- Unit tests for notes reader functionality
- Mock database for testing without real Notes data
- Integration tests with sample Notes database
- Validation of MCP protocol compliance

## Future Enhancements
- Support for note creation/modification (if needed)
- Full-text search indexing for better performance
- Export functionality to various formats
- Backup and sync capabilities
- Web interface for note management