Here's Method 1 as a standalone script:

## ping_ips.sh
```bash
#!/bin/bash

# Color definitions
RED='\033[1;31m'
GREEN='\033[1;32m'
BLUE='\033[1;34m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Check if input file is provided
if [ $# -eq 0 ]; then
    echo -e "${RED}Usage: $0 <ip_file>${NC}"
    echo -e "${YELLOW}Example: $0 ips.txt${NC}"
    exit 1
fi

IP_FILE="$1"
OUTPUT_FILE="ping_results.txt"

# Check if input file exists
if [ ! -f "$IP_FILE" ]; then
    echo -e "${RED}Error: File '$IP_FILE' not found!${NC}"
    exit 1
fi

# Clear previous results
> "$OUTPUT_FILE"

echo -e "${BLUE}Starting ping tests...${NC}"
echo -e "${YELLOW}Results will be saved to: $OUTPUT_FILE${NC}"
echo -e "${YELLOW}Press Ctrl+C to stop at any time${NC}"
echo

# Counter for statistics
total_ips=0
successful_ips=0
failed_ips=0

# Process each IP
while read -r ip; do
    # Skip empty lines and comments
    ip=$(echo "$ip" | sed 's/#.*//' | xargs)
    if [[ -z "$ip" ]]; then
        continue
    fi
    
    ((total_ips++))
    
    echo -e "${BLUE}=== Pinging: $ip ===${NC}" | tee -a "$OUTPUT_FILE"
    
    # Ping the IP twice and process output
    ping_result=$(ping -c 2 -W 2 "$ip" 2>&1)
    ping_exit_code=$?
    
    # Process each line of ping output
    while IFS= read -r line; do
        if [[ "$line" == *"bytes from"* ]]; then
            echo -e "${GREEN}$line${NC}" | tee -a "$OUTPUT_FILE"
        elif [[ "$line" == *"100% packet loss"* ]] || \
             [[ "$line" == *"unreachable"* ]] || \
             [[ "$line" == *"Unknown host"* ]] || \
             [[ "$line" == *"Network is unreachable"* ]]; then
            echo -e "${RED}$line${NC}" | tee -a "$OUTPUT_FILE"
        elif [[ "$line" == *"ping: "* ]] && [ $ping_exit_code -ne 0 ]; then
            echo -e "${RED}$line${NC}" | tee -a "$OUTPUT_FILE"
        else
            echo "$line" | tee -a "$OUTPUT_FILE"
        fi
    done <<< "$ping_result"
    
    # Check if ping was successful
    if [ $ping_exit_code -eq 0 ]; then
        ((successful_ips++))
        echo -e "${GREEN}✓ Ping successful${NC}" | tee -a "$OUTPUT_FILE"
    else
        ((failed_ips++))
        echo -e "${RED}✗ Ping failed${NC}" | tee -a "$OUTPUT_FILE"
    fi
    
    echo | tee -a "$OUTPUT_FILE"
    
done < "$IP_FILE"

# Print summary
echo -e "${BLUE}=== Ping Summary ===${NC}" | tee -a "$OUTPUT_FILE"
echo -e "${GREEN}Successful: $successful_ips${NC}" | tee -a "$OUTPUT_FILE"
echo -e "${RED}Failed: $failed_ips${NC}" | tee -a "$OUTPUT_FILE"
echo -e "${BLUE}Total IPs: $total_ips${NC}" | tee -a "$OUTPUT_FILE"

if [ $successful_ips -eq $total_ips ] && [ $total_ips -gt 0 ]; then
    echo -e "${GREEN}✓ All pings successful!${NC}" | tee -a "$OUTPUT_FILE"
elif [ $failed_ips -eq $total_ips ] && [ $total_ips -gt 0 ]; then
    echo -e "${RED}✗ All pings failed!${NC}" | tee -a "$OUTPUT_FILE"
fi
```

## How to use:

1. **Save the script** to a file (e.g., `ping_ips.sh`)
2. **Make it executable**:
   ```bash
   chmod +x ping_ips.sh
   ```
3. **Create your IP file** (e.g., `ips.txt`):
   ```bash
   cat > ips.txt << EOF
   8.8.8.8
   1.1.1.1
   192.168.1.1
   google.com
   # This is a comment
   10.0.0.1
   EOF
   ```
4. **Run the script**:
   ```bash
   ./ping_ips.sh ips.txt
   ```

## Features:
- ✅ Colored output (blue headers, green success, red failures)
- ✅ Pings each IP twice
- ✅ Output to both console and file (`ping_results.txt`)
- ✅ Handles comments in IP file (lines starting with `#`)
- ✅ Skips empty lines
- ✅ Provides summary statistics
- ✅ Handles various error conditions
- ✅ Timeout of 2 seconds per ping
- ✅ Clean formatting with separators

The script will show colored output in your terminal while simultaneously saving all results to `ping_results.txt`.
