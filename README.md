# SocialCalc Model Context Protocol (MCP) Server

A Model Context Protocol (MCP) server that provides cell-level and sheet-level spreadsheet editing and analysis capabilities for SocialCalc workbooks. It allows LLMs and desktop AI environments (such as Claude Desktop, Cursor, VS Code, and Windsurf) to programmatically inspect, query, format, and edit spreadsheet grids.

This server supports **dual transport modes**: standard MCP Stdio transport for local agent orchestration, and an HTTP JSON-RPC bridge for browser-based tester interfaces and lightweight web clients.

---

## Installation & Setup

You can run this server directly via `npx` or install it locally.

### 1. Running via npx (Recommended for AI Clients)
```bash
npx socialcalc-mcp
```

### 2. Manual Installation
```bash
git clone https://github.com/anisharma07/Socialcalc-AI.git
cd Socialcalc-AI/Socialcalc-MCP
npm install
npm run build
```

---

## Configuration in AI Editors

### Claude Desktop
Add this to your `claude_desktop_config.json` (usually located at `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "socialcalc-mcp": {
      "command": "npx",
      "args": ["-y", "socialcalc-mcp"]
    }
  }
}
```

### Cursor / VS Code / Windsurf
Configure a new MCP server in the editor settings:
- **Type**: `command`
- **Command**: `npx -y socialcalc-mcp`

---

## Tool API Reference

The server exposes 35 tools grouped into functional categories:

### 1. Workbook Inspection & Navigation
- **`list_workbooks`**: Lists all workbook files in a target directory.
  - `directoryPath` (string, optional): Search path (defaults to `./data`).
- **`list_sheets`**: Lists all sheet names and metadata inside a workbook file.
  - `workbookPath` (string, required).
- **`read_sheet`**: Reads the entire content of a worksheet, returning a Markdown table and raw cell details.
  - `workbookPath` (string, required), `sheetNumber` (number, optional), `sheetName` (string, optional).
- **`read_range`**: Reads a coordinate range (e.g. `'A1:C10'`) from a sheet and returns it formatted as a Markdown table.
  - `workbookPath` (string, required), `sheetNumber` (number, optional), `sheetName` (string, optional), `range` (string, required).
- **`get_sheet_dimensions`**: Returns the row and column dimensions (count) of a worksheet.
  - `workbookPath` (string, required), `sheetNumber` (number, optional), `sheetName` (string, optional).
- **`describe_sheet`**: Lists sheet details, column names, column headers contents, and size.
  - `workbookPath` (string, required), `sheetNumber` (number, optional), `sheetName` (string, optional).
- **`summarize_workbook`**: Summarizes the entire workbook, listing all sheets, dimensions, hidden status, and cell counts.
  - `workbookPath` (string, required).
- **`summarize_sheet`**: Provides a structured mathematical and textual summary of the sheet's populated cells.
  - `workbookPath` (string, required), `sheetNumber` (number, optional), `sheetName` (string, optional).

### 2. Structural & Workbook Modification
- **`new_workbook`**: Creates a new empty workbook file at the specified path with an initial sheet.
  - `workbookPath` (string, required), `sheetName` (string, optional).
- **`new_sheet`**: Adds a new empty worksheet to the workbook.
  - `workbookPath` (string, required), `sheetName` (string, required).
- **`rename_sheet`**: Renames an existing worksheet.
  - `workbookPath` (string, required), `oldName` (string, required), `newName` (string, required).
- **`insert_row`**: Inserts empty row(s) before or after a target row letter/index.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `row` (number, required), `count` (number, optional), `position` (`"before" | "after"`, optional).
- **`insert_col`**: Inserts empty column(s) before or after a target column letter.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `column` (string, required), `count` (number, optional), `position` (`"before" | "after"`, optional).
- **`delete_row`**: Deletes one or more rows starting from a target index.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `row` (number, required), `count` (number, optional).
- **`delete_col`**: Deletes one or more columns starting from a target letter.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `column` (string, required), `count` (number, optional).
- **`set_col_width`**: Sets column width in pixels.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `column` (string, required), `width` (number, required).

### 3. Editing & Styling
- **`write_range`**: Writes a single value or a 2D array of data starting at a cell range. Supports auto-calculated formulas.
  - `workbookPath` (string, required), `sheetNumber`/`sheetName` (optional), `range` (string, required), `value` (string, optional), `data` (array, optional).
- **`delete_text`**: Clears cell values/formulas while preserving styling.
  - `workbookPath` (string, required), `range` (string, required).
- **`set_cell_to_default`**: Completely resets cell content, formula, and layout to defaults.
  - `workbookPath` (string, required), `range` (string, required).
- **`set_font_color`**: Applies text foreground color (e.g. `rgb(255,0,0)`, `#ff0000`).
  - `workbookPath` (string, required), `range` (string, required), `color` (string, required).
- **`set_cell_bg`**: Applies cell background color.
  - `workbookPath` (string, required), `range` (string, required), `color` (string, required).
- **`set_border`**: Applies border shorthands or TRBL overrides (e.g. `1px solid rgb(0,0,0)`).
  - `workbookPath` (string, required), `range` (string, required), `border` (string, optional), `top`/`right`/`bottom`/`left` (string, optional).
- **`set_padding`**: Applies cell padding shorthand or TRBL overrides (e.g. `5px 10px`).
  - `workbookPath` (string, required), `range` (string, required), `padding` (string, optional), `top`/`right`/`bottom`/`left` (string, optional).
- **`set_alignment`**: Sets horizontal (`left`/`center`/`right`) and vertical (`top`/`middle`/`bottom`) alignments.
  - `workbookPath` (string, required), `range` (string, required), `align`/`verticalAlign` (string, optional).
- **`set_font_size`**: Sets font size (e.g. `12pt`, `14px`).
  - `workbookPath` (string, required), `range` (string, required), `size` (string/number, required).
- **`set_font_family`**: Sets font family (e.g. `Arial`, `Courier New`).
  - `workbookPath` (string, required), `range` (string, required), `family` (string, required).
- **`set_font_style`**: Sets style (`italic`/`normal`) and weight (`bold`/`normal`).
  - `workbookPath` (string, required), `range` (string, required), `style` (string, optional), `bold` (boolean, optional).
- **`set_format`**: Applies text/numeric formatting formats (`HTML`, `Plain Text`, percentage `0.00%`, currency `$#,##0.00`).
  - `workbookPath` (string, required), `range` (string, required), `format` (string, required).
- **`merge_cells`**: Merges cell ranges.
  - `workbookPath` (string, required), `range` (string, required).
- **`unmerge_cells`**: Restores merged cell ranges.
  - `workbookPath` (string, required), `range` (string, required).

### 4. Formulas Reference & Help
- **`get_formulas`**: Lists all available spreadsheet formulas in SocialCalc, optionally filtered by category.
  - `category` (string, optional).
- **`describe_formula`**: Explains and provides a detailed example of a specific formula.
  - `name` (string, required).
- **`list_tool`**: Lists all registered tools.
- **`describe_tool`**: Describes tool parameters dynamically.
  - `toolName` (string, required).

---

## License

MIT
