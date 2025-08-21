Here are two functions to write and read JSON files:

```python
import json
from pathlib import Path
from typing import Any, Dict
import os

def write_json_to_file(data: Dict[str, Any], filename: str, indent: int = 4, ensure_ascii: bool = False) -> None:
    """
    Write a dictionary to a JSON file.
    
    Args:
        data: Dictionary to write to JSON file
        filename: Name of the output JSON file
        indent: Number of spaces for indentation (default: 4)
        ensure_ascii: If False, allows non-ASCII characters (default: False)
    
    Raises:
        TypeError: If data is not JSON serializable
        IOError: If file cannot be written
    """
    try:
        # Create directory if it doesn't exist
        file_path = Path(filename)
        file_path.parent.mkdir(parents=True, exist_ok=True)
        
        with open(filename, 'w', encoding='utf-8') as json_file:
            json.dump(data, json_file, indent=indent, ensure_ascii=ensure_ascii, sort_keys=True)
        
        print(f"JSON file successfully created: {filename}")
        
    except TypeError as e:
        raise TypeError(f"Data is not JSON serializable: {e}")
    except IOError as e:
        raise IOError(f"Error writing to file {filename}: {e}")
    except Exception as e:
        raise Exception(f"Unexpected error: {e}")

def read_json_from_file(filename: str) -> Dict[str, Any]:
    """
    Read a JSON file into a dictionary.
    
    Args:
        filename: Name of the JSON file to read
        
    Returns:
        Dictionary containing the JSON data
        
    Raises:
        FileNotFoundError: If the file doesn't exist
        json.JSONDecodeError: If the file contains invalid JSON
        IOError: If file cannot be read
    """
    try:
        # Check if file exists
        if not os.path.exists(filename):
            raise FileNotFoundError(f"File not found: {filename}")
        
        with open(filename, 'r', encoding='utf-8') as json_file:
            data = json.load(json_file)
        
        print(f"JSON file successfully loaded: {filename}")
        return data
        
    except FileNotFoundError as e:
        raise FileNotFoundError(f"File not found: {e}")
    except json.JSONDecodeError as e:
        raise json.JSONDecodeError(f"Invalid JSON format in {filename}: {e}", e.doc, e.pos)
    except IOError as e:
        raise IOError(f"Error reading file {filename}: {e}")
    except Exception as e:
        raise Exception(f"Unexpected error: {e}")

# Example usage and demonstration
def demonstrate_functions():
    """Demonstrate how to use the JSON read/write functions."""
    
    # Sample data
    sample_data = {
        "name": "John Doe",
        "age": 30,
        "city": "New York",
        "hobbies": ["reading", "swimming", "coding"],
        "is_student": False,
        "contact": {
            "email": "john.doe@example.com",
            "phone": "+1-555-0123"
        },
        "languages": ["English", "Spanish"]
    }
    
    # File name
    json_filename = "user_data.json"
    
    try:
        # Write JSON to file
        print("=== Writing JSON to file ===")
        write_json_to_file(sample_data, json_filename, indent=2)
        
        # Read JSON from file
        print("\n=== Reading JSON from file ===")
        loaded_data = read_json_from_file(json_filename)
        
        # Display the loaded data
        print("\nLoaded data:")
        print(json.dumps(loaded_data, indent=2))
        
        # Verify data integrity
        print("\n=== Data verification ===")
        print(f"Original and loaded data are identical: {sample_data == loaded_data}")
        
        # Show file info
        file_size = os.path.getsize(json_filename)
        print(f"File size: {file_size} bytes")
        print(f"File location: {os.path.abspath(json_filename)}")
        
    except Exception as e:
        print(f"Error: {e}")

# Additional utility function
def update_json_file(filename: str, updates: Dict[str, Any]) -> None:
    """
    Update an existing JSON file with new data.
    
    Args:
        filename: Name of the JSON file to update
        updates: Dictionary with updates (will be merged with existing data)
    """
    try:
        # Read existing data
        existing_data = read_json_from_file(filename)
        
        # Update with new data
        existing_data.update(updates)
        
        # Write back to file
        write_json_to_file(existing_data, filename)
        print(f"JSON file updated: {filename}")
        
    except Exception as e:
        print(f"Error updating file: {e}")

if __name__ == "__main__":
    # Run demonstration
    demonstrate_functions()
    
    # Example of updating a JSON file
    print("\n=== Updating JSON file ===")
    try:
        update_json_file("user_data.json", {"last_updated": "2024-01-15", "status": "active"})
        
        # Read to verify update
        updated_data = read_json_from_file("user_data.json")
        print("Updated data keys:", list(updated_data.keys()))
        
    except Exception as e:
        print(f"Update example failed: {e}")
```

## Usage Examples:

### Basic usage:
```python
# Write data
data = {"key": "value", "number": 42}
write_json_to_file(data, "output.json")

# Read data
loaded_data = read_json_from_file("output.json")
print(loaded_data)
```

### With custom settings:
```python
# Write with custom indentation and allow non-ASCII
data = {"name": "José", "message": "¡Hola!"}
write_json_to_file(data, "spanish.json", indent=2, ensure_ascii=False)

# Read it back
result = read_json_from_file("spanish.json")
```

### Error handling:
```python
try:
    data = read_json_from_file("nonexistent.json")
except FileNotFoundError as e:
    print(f"File not found: {e}")
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
```

## Features:

1. **Error Handling**: Comprehensive error handling for common issues
2. **Type Hints**: Proper type annotations for better code documentation
3. **UTF-8 Support**: Proper encoding handling for international characters
4. **Directory Creation**: Automatically creates parent directories if needed
5. **File Existence Check**: Verifies file exists before reading
6. **Pretty Printing**: Clean, readable JSON output with indentation
7. **Data Verification**: Example shows how to verify data integrity

The functions are robust and handle most common edge cases you might encounter when working with JSON files.
