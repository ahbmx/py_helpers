Based on your space_key and page ID, here are several ways to list all pages in a space:

## Method 1: List All Pages in a Space (Using space_key)

```python
import requests
from requests.auth import HTTPBasicAuth

def list_all_pages_in_space(confluence_url, username, password, space_key):
    """List all pages in a specific space"""
    all_pages = []
    start = 0
    limit = 100
    
    while True:
        url = f"{confluence_url}/rest/api/content"
        params = {
            "spaceKey": space_key,
            "type": "page",
            "expand": "space,version,history",
            "start": start,
            "limit": limit
        }
        
        response = requests.get(
            url,
            params=params,
            auth=HTTPBasicAuth(username, password)
        )
        
        if response.status_code != 200:
            print(f"Error: {response.status_code} - {response.text}")
            break
            
        data = response.json()
        pages = data["results"]
        all_pages.extend(pages)
        
        # Check if we have more pages
        if len(pages) < limit or not data.get("_links", {}).get("next"):
            break
            
        start += limit
    
    return all_pages

# Usage
confluence_url = "http://your-confluence-server"
username = "your-username"
password = "your-password"
space_key = "YOUR_SPACE_KEY"  # Replace with your space key

pages = list_all_pages_in_space(confluence_url, username, password, space_key)

print(f"Found {len(pages)} pages in space '{space_key}':")
for i, page in enumerate(pages, 1):
    print(f"{i}. {page['title']}")
    print(f"   ID: {page['id']}")
    print(f"   Created: {page['history']['createdDate']}")
    print(f"   Version: {page['version']['number']}")
    print(f"   URL: {confluence_url}{page['_links']['webui']}")
    print()
```

## Method 2: Get Page Hierarchy (Tree Structure)

```python
import requests
from requests.auth import HTTPBasicAuth

def get_page_hierarchy(confluence_url, username, password, space_key):
    """Get pages organized in a hierarchical structure"""
    def get_child_pages(parent_id=None, level=0):
        params = {
            "spaceKey": space_key,
            "type": "page",
            "expand": "children.page",
            "limit": 100
        }
        
        if parent_id:
            params["parentId"] = parent_id
        
        response = requests.get(
            f"{confluence_url}/rest/api/content",
            params=params,
            auth=HTTPBasicAuth(username, password)
        )
        
        if response.status_code != 200:
            return []
            
        pages = response.json()["results"]
        result = []
        
        for page in pages:
            page_info = {
                "id": page["id"],
                "title": page["title"],
                "level": level,
                "children": get_child_pages(page["id"], level + 1)
            }
            result.append(page_info)
        
        return result
    
    return get_child_pages()

# Usage
hierarchy = get_page_hierarchy(confluence_url, username, password, space_key)

def print_hierarchy(pages, indent=0):
    for page in pages:
        print("  " * indent + f"- {page['title']} (ID: {page['id']})")
        print_hierarchy(page["children"], indent + 1)

print("Page Hierarchy:")
print_hierarchy(hierarchy)
```

## Method 3: Using CQL (Confluence Query Language)

```python
import requests
from requests.auth import HTTPBasicAuth

def search_pages_with_cql(confluence_url, username, password, space_key):
    """Search pages using CQL"""
    cql_query = f'space = "{space_key}" and type = "page"'
    
    url = f"{confluence_url}/rest/api/content/search"
    params = {
        "cql": cql_query,
        "expand": "space,version",
        "limit": 1000
    }
    
    response = requests.get(
        url,
        params=params,
        auth=HTTPBasicAuth(username, password)
    )
    
    if response.status_code == 200:
        return response.json()["results"]
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return []

# Usage
pages = search_pages_with_cql(confluence_url, username, password, space_key)

print(f"Found {len(pages)} pages using CQL:")
for page in pages:
    print(f"- {page['title']} (ID: {page['id']})")
```

## Method 4: Get Detailed Page Information

```python
import requests
from requests.auth import HTTPBasicAuth

def get_page_details(confluence_url, username, password, page_id):
    """Get detailed information about a specific page"""
    url = f"{confluence_url}/rest/api/content/{page_id}"
    params = {
        "expand": "space,version,body.view,children.page,ancestors,descendants"
    }
    
    response = requests.get(
        url,
        params=params,
        auth=HTTPBasicAuth(username, password)
    )
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return None

# Usage - get details for a specific page
page_details = get_page_details(confluence_url, username, password, "123456")  # Replace with actual page ID

if page_details:
    print(f"Title: {page_details['title']}")
    print(f"Space: {page_details['space']['name']}")
    print(f"Version: {page_details['version']['number']}")
    print(f"Created: {page_details['version']['when']}")
    print(f"Content: {page_details['body']['view']['value'][:200]}...")  # First 200 chars
```

## Method 5: Export to CSV

```python
import csv
import requests
from requests.auth import HTTPBasicAuth
from datetime import datetime

def export_pages_to_csv(confluence_url, username, password, space_key, filename=None):
    """Export all pages in a space to CSV"""
    if filename is None:
        filename = f"confluence_pages_{space_key}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.csv"
    
    pages = list_all_pages_in_space(confluence_url, username, password, space_key)
    
    with open(filename, 'w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['ID', 'Title', 'Space', 'Version', 'Created Date', 'URL']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        
        writer.writeheader()
        for page in pages:
            writer.writerow({
                'ID': page['id'],
                'Title': page['title'],
                'Space': page['space']['name'],
                'Version': page['version']['number'],
                'Created Date': page['history']['createdDate'],
                'URL': f"{confluence_url}{page['_links']['webui']}"
            })
    
    print(f"Exported {len(pages)} pages to {filename}")

# Usage
export_pages_to_csv(confluence_url, username, password, space_key)
```

## Key Parameters to Use:

- `spaceKey`: Filter by space key
- `type`: Set to "page" to get only pages
- `expand`: Request additional information (space, version, body, history, children, etc.)
- `limit`: Number of results per request (max 100)
- `start`: Pagination offset

Choose the method that best fits your needs. Method 1 is the most straightforward for simply listing all pages in a space.
