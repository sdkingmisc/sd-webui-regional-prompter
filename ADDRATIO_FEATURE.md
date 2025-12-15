# Regional Prompter Prompt-Based Features

## Overview

Regional Prompter now supports two powerful prompt-based features that make configurations completely self-contained:

1. **`ADDRATIO`** - Embed grid layout ratios directly in prompts
2. **`RPENABLE`** - Activate and configure Regional Prompter entirely through prompts

These features eliminate the need for UI configuration and make prompts fully shareable and API-friendly.

## Syntax

```
ADDRATIO[ratio_specification] your_prompt_content_here
```

### Ratio Specification Format

- **Columns only**: `1,2,1` (creates 3 columns with ratios 1:2:1)
- **Rows and columns**: `1,2;3,1` (creates 2 rows with ratios 1:2, first row has 2 columns with ratios 3:1)
- **Complex 2D grids**: `1,2,1;2,3,2;1,1,1` (creates 3 rows with different column configurations)

The format follows the same rules as the UI "Divide Ratio" field:
- `,` separates columns within a row
- `;` separates rows
- Numbers represent relative proportions

## Examples

### Example 1: Simple 3-Column Layout
```
ADDRATIO[1,1,1] (blue sky:1.2) ADDCOL green hair twintail ADDCOL red dress
```
This creates three equal columns (33.3% each).

### Example 2: Unequal Columns
```
ADDRATIO[3,1,2] landscape ADDCOL character ADDCOL background details
```
This creates three columns with ratios 3:1:2 (50%, 16.7%, 33.3%).

### Example 3: 2D Grid Layout
```
ADDRATIO[1,2,1;2,3,2] (blue sky:1.2) ADDCOL green hair twintail ADDCOL (aquarium:1.3) ADDROW (messy desk:1.2) ADDCOL orange dress and sofa
```
This creates:
- First row: 3 columns with ratios 1:2:1
- Second row: 2 columns with ratios 2:3:2

### Example 4: With Base Prompt
```
ADDRATIO[2,1,1] masterpiece, best quality ADDBASE red hair ADDCOL blue eyes ADDCOL green dress
```
The base prompt "masterpiece, best quality" applies to all regions with the specified column ratios.

### Example 5: With Common Prompt
```
ADDRATIO[1,1;2,1] fantasy landscape ADDCOMM castle ADDCOL mountains ADDROW forest ADDCOL river
```
The common prompt "fantasy landscape" is added to all regions.

## Features

### Case Insensitive
Both `ADDRATIO[1,2,1]` and `addratio[1,2,1]` work.

### Position Flexible
The ADDRATIO can be placed anywhere in the prompt:
```
(blue sky:1.2) ADDCOL ADDRATIO[1,2,1] green hair ADDCOL red dress
```

### Override UI Settings
When ADDRATIO is present in the prompt, it completely overrides any ratios set in the Regional Prompter UI.

### Backward Compatible
- Prompts without ADDRATIO continue to use UI ratios as before
- All existing functionality remains unchanged

## Technical Details

### Processing Order
1. ADDRATIO is extracted from the prompt during ratio processing
2. Extracted ratios override UI "Divide Ratio" settings
3. ADDRATIO syntax is removed from the prompt before further processing
4. Normal Regional Prompter processing continues with the extracted ratios

### Supported Modes
- **Matrix Mode**: Full support - ratios control grid layout
- **Mask Mode**: ADDRATIO is extracted but ignored (masks define regions)
- **Prompt Mode**: ADDRATIO is extracted but not used (prompt-based region detection)

### Debug Information
When debug mode is enabled, you'll see:
```
Using ratios from prompt: 1,2,1;2,3,2
```

## Migration Guide

### Converting UI-based setups to ADDRATIO

**Before** (UI + Prompt):
- UI Divide Ratio: `1,2,1;2,3,2`
- Prompt: `(blue sky:1.2) ADDCOL green hair ADDCOL (aquarium:1.3) ADDROW (desk:1.2) ADDCOL orange dress`

**After** (Self-contained):
- Prompt: `ADDRATIO[1,2,1;2,3,2] (blue sky:1.2) ADDCOL green hair ADDCOL (aquarium:1.3) ADDROW (desk:1.2) ADDCOL orange dress`

## Error Handling

- Invalid ratio syntax falls back to equal proportions
- Missing ADDRATIO uses UI ratios as before
- Malformed brackets are ignored
- Debug mode shows warnings for any issues

## API Usage

When using the API, you can now embed ratios directly in the prompt:

```json
{
  "prompt": "ADDRATIO[1,2,1] red hair ADDCOL blue eyes ADDCOL green dress",
  "alwayson_scripts": {
    "Regional Prompter": {
      "args": [true, false, "Matrix", "Columns", "Mask", "Prompt", "1,1,1", "", false, false, false, "Attention", false, "0", "0", "0", ""]
    }
  }
}
```

The `"1,1,1"` in args[6] will be overridden by `ADDRATIO[1,2,1]` from the prompt.

---

# RPENABLE Feature - Prompt-Based Activation

## Overview

The `RPENABLE` feature allows you to activate and configure Regional Prompter entirely through prompt text, without needing to enable it in the UI or configure API parameters.

## Syntax

### Basic Activation
```
RPENABLE your_prompt_content_here
```

### With Configuration
```
RPENABLE[settings] your_prompt_content_here
```

## Configuration Options

### Mode Settings
- `Matrix` - Matrix/grid mode
- `Mask` - Mask-based regions
- `Prompt` - Prompt mode
- `Prompt-Ex` - Extended prompt mode

### Calculation Mode
- `Attention` - Attention mode (faster)
- `Latent` - Latent mode (slower, more precise)

### Matrix Submode
- `Horizontal` or `Rows` - Horizontal division
- `Vertical` or `Columns` - Vertical division

### Boolean Options
- `usebase=true/false` - Enable/disable base prompt
- `usecom=true/false` - Enable/disable common prompt

## Examples

### Example 1: Simple Activation
```
RPENABLE red hair ADDCOL blue eyes ADDCOL green dress
```
Activates Regional Prompter with default settings.

### Example 2: Matrix Mode with Attention
```
RPENABLE[Matrix,Attention] ADDRATIO[1,2,1] landscape ADDCOL character ADDCOL background
```
Activates with Matrix mode and Attention calculation.

### Example 3: Full Configuration
```
RPENABLE[Matrix,Attention,Vertical,usebase=true] ADDRATIO[2,1,1] masterpiece ADDBASE red hair ADDCOL blue eyes ADDCOL green dress
```
Complete configuration with vertical matrix, attention mode, and base prompt enabled.

### Example 4: Prompt Mode
```
RPENABLE[Prompt,Latent,usecom=true] fantasy landscape ADDCOMM castle BREAK mountains BREAK forest BREAK river
```
Activates prompt mode with latent calculation and common prompt.

### Example 5: Key-Value Configuration
```
RPENABLE[mode=Matrix,calcmode=Attention,submode=Horizontal,usebase=true] ADDRATIO[1,1,1] sky ADDCOL character ADDCOL ground
```
Explicit key-value configuration syntax.

## Features

### Auto-Activation
- **RPENABLE detection**: Automatically activates Regional Prompter when found
- **ADDRATIO detection**: Also auto-activates when `ADDRATIO[...]` is found
- **UI override**: Works regardless of UI checkbox state

### Case Insensitive
Both `RPENABLE[Matrix]` and `rpenable[matrix]` work.

### Position Flexible
RPENABLE can be placed anywhere in the prompt:
```
red hair ADDCOL RPENABLE[Matrix] blue eyes ADDCOL green dress
```

### Combined Features
```
RPENABLE[Matrix,Attention] ADDRATIO[1,2,1] red hair ADDCOL blue eyes ADDCOL green dress
```
Both activation and ratio configuration in one prompt.

## API Usage Transformation

### Before (Complex API Configuration)
```json
{
  "prompt": "red hair ADDCOL blue eyes ADDCOL green dress",
  "alwayson_scripts": {
    "Regional Prompter": {
      "args": [true, false, "Matrix", "Vertical", "Mask", "Prompt", "1,2,1", "", false, true, false, "Attention", false, "0", "0", "0", ""]
    }
  }
}
```

### After (Simple Prompt-Based)
```json
{
  "prompt": "RPENABLE[Matrix,Attention,Vertical,usebase=true] ADDRATIO[1,2,1] red hair ADDCOL blue eyes ADDCOL green dress"
}
```

## Processing Order

1. **RPENABLE extraction**: Settings extracted and applied to override UI/API parameters
2. **Auto-activation**: Regional Prompter activated if RPENABLE or ADDRATIO found
3. **ADDRATIO processing**: Ratios extracted and applied
4. **Keyword cleanup**: RPENABLE and ADDRATIO removed from prompts
5. **Normal processing**: Regional Prompter continues with extracted settings

## Supported Modes

- **Matrix Mode**: Full support with ratio and submode configuration
- **Mask Mode**: Activation works, configuration noted but regions defined by masks
- **Prompt Mode**: Full support with threshold and ratio configuration

## Debug Information

When debug mode is enabled, you'll see:
```
RPENABLE settings applied: {'mode': 'Matrix', 'calcmode': 'Attention'}
Regional Prompter auto-activated by prompt
```

## Migration Examples

### UI-Based to Prompt-Based

**Before** (UI Configuration):
- Enable Regional Prompter checkbox
- Set Mode: Matrix
- Set Submode: Vertical
- Set Calculation: Attention
- Set Ratios: 1,2,1
- Prompt: `red hair ADDCOL blue eyes ADDCOL green dress`

**After** (Self-Contained):
```
RPENABLE[Matrix,Attention,Vertical] ADDRATIO[1,2,1] red hair ADDCOL blue eyes ADDCOL green dress
```

### API Simplification

**Before** (17 parameters):
```json
"args": [true, false, "Matrix", "Vertical", "Mask", "Prompt", "1,2,1", "", false, true, false, "Attention", false, "0", "0", "0", ""]
```

**After** (0 parameters):
```json
"prompt": "RPENABLE[Matrix,Attention,Vertical,usebase=true] ADDRATIO[1,2,1] ..."
```

## Error Handling

- **Invalid settings**: Ignored, falls back to defaults
- **Missing RPENABLE**: Uses UI/API settings as before
- **Malformed syntax**: Gracefully ignored
- **Conflicting settings**: Prompt settings take precedence

## Backward Compatibility

- **Existing prompts**: Continue to work unchanged
- **UI controls**: Still functional when RPENABLE not present
- **API parameters**: Still respected when RPENABLE not used
- **Mixed usage**: RPENABLE overrides UI/API when present

This feature makes Regional Prompter configurations completely portable and eliminates the complexity of API parameter management.
