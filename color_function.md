Here's a Python function that accepts strings, lists, or dictionaries and prints them with colored output without using the typing module:

```python
# ANSI color codes
COLORS = {
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
    'reset': '\033[0m'
}

def print_colored(data, color='white', indent=2):
    """
    Print data with colored output.
    
    Parameters:
    - data: string, list, or dictionary to print
    - color: color name from available colors (default: 'white')
    - indent: indentation level for nested structures (default: 2)
    """
    # Get the ANSI color code
    color_code = COLORS.get(color.lower(), COLORS['white'])
    reset_code = COLORS['reset']
    
    # Handle different data types
    if isinstance(data, str):
        # Print string directly
        print(f"{color_code}{data}{reset_code}")
    
    elif isinstance(data, list):
        # Print list with formatting
        print(f"{color_code}[{reset_code}")
        for i, item in enumerate(data):
            if isinstance(item, (list, dict)):
                # Recursively handle nested structures
                print_colored(item, color, indent + 2)
            else:
                print(f"{' ' * indent}{color_code}{item}{reset_code}")
            if i < len(data) - 1:
                print(f"{' ' * indent}{color_code},{reset_code}")
        print(f"{color_code}]{reset_code}")
    
    elif isinstance(data, dict):
        # Print dictionary with formatting
        print(f"{color_code}{'{'}{reset_code}")
        items = list(data.items())
        for i, (key, value) in enumerate(items):
            if isinstance(value, (list, dict)):
                print(f"{' ' * indent}{color_code}{key}: {reset_code}")
                print_colored(value, color, indent + 2)
            else:
                print(f"{' ' * indent}{color_code}{key}: {value}{reset_code}")
            if i < len(items) - 1:
                print(f"{' ' * indent}{color_code},{reset_code}")
        print(f"{color_code}{'}'}{reset_code}")
    
    else:
        # Handle other types by converting to string
        print(f"{color_code}{str(data)}{reset_code}")

# Example usage and demonstration
if __name__ == "__main__":
    # Test with different data types
    print("=== String Example ===")
    print_colored("Hello, World!", "green")
    
    print("\n=== List Example ===")
    sample_list = ["apple", "banana", "cherry", [1, 2, 3]]
    print_colored(sample_list, "blue")
    
    print("\n=== Dictionary Example ===")
    sample_dict = {
        "name": "John",
        "age": 30,
        "hobbies": ["reading", "swimming", "coding"],
        "address": {
            "street": "123 Main St",
            "city": "New York"
        }
    }
    print_colored(sample_dict, "magenta")
    
    print("\n=== Available Colors ===")
    for color_name in COLORS:
        if color_name != 'reset':
            print_colored(f"This is {color_name} color", color_name)
```

This function provides:

1. **Multiple color options**: 16 different ANSI colors including bright variants
2. **Flexible input**: Accepts strings, lists, dictionaries, or any other type
3. **Formatted output**: Pretty-prints lists and dictionaries with proper indentation
4. **Recursive handling**: Properly handles nested lists and dictionaries
5. **Error handling**: Falls back to white color if an invalid color is specified

**Available colors**: `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, `bright_black`, `bright_red`, `bright_green`, `bright_yellow`, `bright_blue`, `bright_magenta`, `bright_cyan`, `bright_white`

**Usage examples:**
```python
print_colored("Hello!", "red")
print_colored([1, 2, 3], "green")
print_colored({"key": "value"}, "blue")
```
