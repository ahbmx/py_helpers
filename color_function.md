Here's the enhanced function with both text colors and background colors:

```python
# ANSI color codes
COLORS = {
    # Text colors
    'black': '\033[30m',
    'red': '\033[31m',
    'green': '\033[32m',
    'yellow': '\033[33m',
    'blue': '\033[34m',
    'magenta': '\033[35m',
    'cyan': '\033[36m',
    'white': '\033[37m',
    'bright_black': '\033[90m',
    'bright_red': '\033[91m',
    'bright_green': '\033[92m',
    'bright_yellow': '\033[93m',
    'bright_blue': '\033[94m',
    'bright_magenta': '\033[95m',
    'bright_cyan': '\033[96m',
    'bright_white': '\033[97m',
    
    # Background colors
    'bg_black': '\033[40m',
    'bg_red': '\033[41m',
    'bg_green': '\033[42m',
    'bg_yellow': '\033[43m',
    'bg_blue': '\033[44m',
    'bg_magenta': '\033[45m',
    'bg_cyan': '\033[46m',
    'bg_white': '\033[47m',
    'bg_bright_black': '\033[100m',
    'bg_bright_red': '\033[101m',
    'bg_bright_green': '\033[102m',
    'bg_bright_yellow': '\033[103m',
    'bg_bright_blue': '\033[104m',
    'bg_bright_magenta': '\033[105m',
    'bg_bright_cyan': '\033[106m',
    'bg_bright_white': '\033[107m',
    
    # Text styles
    'bold': '\033[1m',
    'dim': '\033[2m',
    'italic': '\033[3m',
    'underline': '\033[4m',
    'blink': '\033[5m',
    'reverse': '\033[7m',
    'hidden': '\033[8m',
    
    'reset': '\033[0m'
}

def print_colored(data, color=None, bg_color=None, style=None, indent=2):
    """
    Print data with colored output and optional background color and styles.
    
    Parameters:
    - data: string, list, or dictionary to print
    - color: text color name from available colors
    - bg_color: background color name (prefix with 'bg_')
    - style: text style (bold, underline, italic, etc.)
    - indent: indentation level for nested structures (default: 2)
    """
    # Build the ANSI escape sequence
    escape_sequence = ""
    
    if color:
        color_code = COLORS.get(color.lower(), '')
        if color_code:
            escape_sequence += color_code
    
    if bg_color:
        bg_code = COLORS.get(bg_color.lower(), '')
        if not bg_code and bg_color.lower().startswith('bg_'):
            bg_code = COLORS.get(bg_color.lower(), '')
        if not bg_code and not bg_color.lower().startswith('bg_'):
            bg_code = COLORS.get('bg_' + bg_color.lower(), '')
        if bg_code:
            escape_sequence += bg_code
    
    if style:
        style_code = COLORS.get(style.lower(), '')
        if style_code:
            escape_sequence += style_code
    
    reset_code = COLORS['reset']
    
    # Handle different data types
    if isinstance(data, str):
        # Print string directly
        print(f"{escape_sequence}{data}{reset_code}")
    
    elif isinstance(data, list):
        # Print list with formatting
        print(f"{escape_sequence}[{reset_code}")
        for i, item in enumerate(data):
            if isinstance(item, (list, dict)):
                # Recursively handle nested structures
                print_colored(item, color, bg_color, style, indent + 2)
            else:
                print(f"{' ' * indent}{escape_sequence}{item}{reset_code}")
            if i < len(data) - 1:
                print(f"{' ' * indent}{escape_sequence},{reset_code}")
        print(f"{escape_sequence}]{reset_code}")
    
    elif isinstance(data, dict):
        # Print dictionary with formatting
        print(f"{escape_sequence}{'{'}{reset_code}")
        items = list(data.items())
        for i, (key, value) in enumerate(items):
            if isinstance(value, (list, dict)):
                print(f"{' ' * indent}{escape_sequence}{key}: {reset_code}")
                print_colored(value, color, bg_color, style, indent + 2)
            else:
                print(f"{' ' * indent}{escape_sequence}{key}: {value}{reset_code}")
            if i < len(items) - 1:
                print(f"{' ' * indent}{escape_sequence},{reset_code}")
        print(f"{escape_sequence}{'}'}{reset_code}")
    
    else:
        # Handle other types by converting to string
        print(f"{escape_sequence}{str(data)}{reset_code}")

# Helper function to show available options
def show_color_options():
    """Display all available color and style options"""
    print("=== TEXT COLORS ===")
    text_colors = [c for c in COLORS.keys() if not c.startswith('bg_') and c not in ['reset', 'bold', 'dim', 'italic', 'underline', 'blink', 'reverse', 'hidden']]
    for color in text_colors:
        print_colored(f"  {color}", color)
    
    print("\n=== BACKGROUND COLORS ===")
    bg_colors = [c for c in COLORS.keys() if c.startswith('bg_')]
    for bg_color in bg_colors:
        print_colored(f"  {bg_color}", bg_color=bg_color)
    
    print("\n=== TEXT STYLES ===")
    styles = ['bold', 'dim', 'italic', 'underline', 'blink', 'reverse', 'hidden']
    for style in styles:
        print_colored(f"  {style}", style=style)

# Example usage and demonstration
if __name__ == "__main__":
    # Show available options
    show_color_options()
    
    print("\n" + "="*50)
    print("EXAMPLES:")
    print("="*50)
    
    # Test with different combinations
    print("\n=== String with text color ===")
    print_colored("Hello, World!", "green")
    
    print("\n=== String with background color ===")
    print_colored("Warning message!", bg_color="bg_red")
    
    print("\n=== String with both text and background ===")
    print_colored("Important notice!", "white", "bg_blue")
    
    print("\n=== String with style ===")
    print_colored("Bold and underlined", "yellow", style="bold")
    print_colored("Italic text", "cyan", style="italic")
    
    print("\n=== List with colored background ===")
    sample_list = ["apple", "banana", "cherry"]
    print_colored(sample_list, bg_color="bg_bright_black")
    
    print("\n=== Dictionary with complex styling ===")
    sample_dict = {
        "name": "John Doe",
        "age": 30,
        "active": True,
        "skills": ["Python", "JavaScript", "SQL"]
    }
    print_colored(sample_dict, "bright_white", "bg_magenta", "bold")
    
    print("\n=== Combined example ===")
    print_colored("SUCCESS!", "bright_green", "bg_black", "bold")
    print_colored("This is a multi-line\nexample with background", "white", "bg_blue")
```

This enhanced version includes:

**New Features:**
- **Background colors**: 16 background colors (e.g., `bg_red`, `bg_blue`, `bg_bright_green`)
- **Text styles**: bold, dim, italic, underline, blink, reverse, hidden
- **Combined styling**: Use text color, background color, and styles together

**Available Options:**
- **Text colors**: 16 options (same as before)
- **Background colors**: 16 options (prefix with `bg_`)
- **Text styles**: bold, dim, italic, underline, blink, reverse, hidden

**Usage examples:**
```python
# Text color only
print_colored("Hello", "red")

# Background color only  
print_colored("Warning", bg_color="bg_yellow")

# Both text and background
print_colored("Error", "white", "bg_red")

# With text style
print_colored("Important", "yellow", style="bold")

# All combined
print_colored("CRITICAL", "bright_white", "bg_red", "bold")
```

The function automatically handles the `bg_` prefix for background colors and combines multiple styling options seamlessly.
