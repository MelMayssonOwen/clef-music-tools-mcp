# Clef Music Tools MCP server

Music theory helpers for piano players: transpose chord charts, spell any chord as piano notes with MIDI numbers, turn guitar tab into piano pitches, and find Clef guides on converting music between formats.

This is a hosted remote MCP server. There is nothing to install and no account or API key.

- Endpoint: `https://clefdrills.com/mcp` (Streamable HTTP)
- Tools: `transpose_chords`, `spell_piano_chord`, `tab_to_piano_notes`, `find_music_guides` (all read-only)
- Docs: https://go.clefdrills.com/mkt-github-docs
- Website: https://go.clefdrills.com/mkt-github
- Server card: https://clefdrills.com/.well-known/mcp/server-card.json
- License: MIT (this repository's metadata and docs)

## Connect

Claude Code:

```bash
claude mcp add --transport http clef-music-tools https://clefdrills.com/mcp
```

Claude (claude.ai and desktop), ChatGPT and other apps with custom connectors: add a custom connector (remote MCP server) and paste https://clefdrills.com/mcp. No authentication.

Cursor (`~/.cursor/mcp.json` or `.cursor/mcp.json`) and any client that reads `mcpServers`:

```json
{
  "mcpServers": {
    "clef-music-tools": {
      "url": "https://clefdrills.com/mcp"
    }
  }
}
```

VS Code (`.vscode/mcp.json`):

```json
{
  "servers": {
    "clef-music-tools": {
      "type": "http",
      "url": "https://clefdrills.com/mcp"
    }
  }
}
```

Test with curl:

```bash
curl -s https://clefdrills.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"1"}}}'
```

## Tools

### transpose_chords

Move a chord chart up or down by semitones, including slash bass notes. Set preferFlats for flat keys.

Input:

```
{ "chords": string (1-10000 chars), "semitones": integer (-24 to 24), "preferFlats": boolean (default false) }
```

Example call:

```bash
curl -s https://clefdrills.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"transpose_chords","arguments":{"chords":"C G/B Am F","semitones":2}}}'
```

Result (structuredContent, shortened where noted):

```json
{ "chords": "D A/C# Bm G", "browserTool": "https://clefdrills.com/tools/chord-transposer/" }
```

### spell_piano_chord

List the notes of one chord symbol as piano pitches with octave and MIDI number. It spells the chord; it does not write an arrangement or rhythm.

Input:

```
{ "symbol": string (1-40 chars), "octave": integer (1 to 6, default 4) }
```

Example call:

```bash
curl -s https://clefdrills.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"spell_piano_chord","arguments":{"symbol":"Cmaj7","octave":4}}}'
```

Result (structuredContent, shortened where noted):

```json
{ "notes": [
    { "midi": 60, "name": "C4", "letter": "C", "accidental": "", "octave": 4 },
    { "midi": 64, "name": "E4", "letter": "E", "accidental": "", "octave": 4 },
    { "midi": 67, "name": "G4", "letter": "G", "accidental": "", "octave": 4 },
    { "midi": 71, "name": "B4", "letter": "B", "accidental": "", "octave": 4 }
  ],
  "browserTool": "https://clefdrills.com/tools/piano-chord-finder/" }
```

### tab_to_piano_notes

Read six-string ASCII guitar tab (standard tuning) as sounding piano pitches, one event per tab column. Tab has no rhythm, so durations are not returned.

Input:

```
{ "tab": string (1-20000 chars), "capo": integer (0 to 12, default 0) }
```

Example call:

```bash
curl -s https://clefdrills.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"tab_to_piano_notes","arguments":{"tab":"e|-0-3-|\nB|-1-0-|\nG|-0-0-|\nD|-2-0-|\nA|-3-2-|\nE|-----|"}}}'
```

Result (structuredContent, shortened where noted):

```json
{ "result": { "events": [
    { "index": 0, "block": 0, "column": 1, "midis": [48, 52, 55, 60, 64], "sources": [ ... ] },
    { "index": 1, "block": 0, "column": 3, "midis": [47, 50, 55, 59, 67], "sources": [ ... ] }
  ], "warnings": ["Columns show order, not rhythm; no note durations are encoded."] },
  "warnings": ["Rhythm and note durations are not inferred from spacing. Standard six-string guitar tuning only."],
  "browserTool": "https://clefdrills.com/tools/guitar-tab-to-piano/" }
```

### find_music_guides

Find Clef guides by words in the title and description. Guides explain workflows; they are not file converters. Read a returned guide with resources/read on its uri.

Input:

```
{ "query": string (1-200 chars) }
```

Example call:

```bash
curl -s https://clefdrills.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"find_music_guides","arguments":{"query":"pdf to midi"}}}'
```

Result (structuredContent, shortened where noted):

```json
{ "guides": [
    { "title": "PDF Sheet Music to MusicXML: Scan, Check and Edit",
      "description": "A practical guide to converting PDF sheet music into editable MusicXML with optical music recognition, followed by systematic notation checks and cleanup.",
      "uri": "https://clefdrills.com/blog/pdf-sheet-music-to-musicxml.html" },
    ... up to 8 guides
  ],
  "directory": "https://clefdrills.com/blog/music-conversion-workflow-finder.html" }
```

## Limits

- The tools do not process audio, PDFs, MusicXML, Guitar Pro files or any uploaded file. Input is text only.
- Guitar tab conversion covers six-string standard tuning and returns pitches, not rhythm.
- Chord spelling covers the chord symbols the Clef chord finder supports; an unsupported symbol returns an error result.
- The server is stateless: no sessions, no streaming, one JSON response per POST.
- Request bodies are limited to 64 KB.

## Privacy

- All four tools are read-only and idempotent. They change nothing and store nothing about you.
- No account, API key or sign-in is needed.
- Inputs are processed in memory to build the response. Chord charts and tabs you send are not saved by the tools.
- See the [Privacy Policy](https://clefdrills.com/privacy/) for how clefdrills.com handles ordinary web traffic.

## Support

Email support@clefdrills.com. Full documentation: https://go.clefdrills.com/mkt-github-docs

## Repository contents

- `server.json`: MCP registry manifest for the remote server.
- `.mcp.json`, `.cursor-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `plugins/clef-music-tools/`: plugin metadata for clients and directories that scan a repository.
- The server itself runs at clefdrills.com; this repository holds its metadata and documentation.
