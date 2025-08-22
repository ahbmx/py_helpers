# Python Function for Pure Storage Capacity Report with SMTP Email

Here's a complete solution that uses username/password authentication for Pure Storage arrays, creates a capacity utilization chart, and sends it via email:

```python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
import matplotlib.pyplot as plt
import numpy as np
from datetime import datetime
import tempfile
import os
from purestorage import FlashArray
import logging

# Set up logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

def get_array_capacity_utilization(array_hosts, usernames, passwords):
    """
    Get capacity and utilization data from multiple Pure Storage arrays using username/password auth
    
    Args:
        array_hosts (list): List of array hostnames/IP addresses
        usernames (list): List of usernames for each array
        passwords (list): List of passwords for each array
    
    Returns:
        list: List of dictionaries with array capacity data
    """
    arrays_data = []
    
    for i, host in enumerate(array_hosts):
        try:
            username = usernames[i] if i < len(usernames) else usernames[0]
            password = passwords[i] if i < len(passwords) else passwords[0]
            
            logger.info(f"Connecting to array: {host}")
            
            # Connect to the array using username/password
            array = FlashArray(host, username=username, password=password)
            
            # Get array information
            array_info = array.get()
            array_name = array_info.get('array_name', host)
            
            # Get space information
            space_info = array.get_space(space=True)
            
            # Calculate utilization
            total_capacity = space_info.get('capacity', 0)
            used_capacity = space_info.get('total', 0)
            utilization_percent = (used_capacity / total_capacity * 100) if total_capacity > 0 else 0
            
            arrays_data.append({
                'array_name': array_name,
                'host': host,
                'total_capacity_gb': total_capacity / 1024 / 1024 / 1024,  # Convert to GB
                'used_capacity_gb': used_capacity / 1024 / 1024 / 1024,    # Convert to GB
                'utilization_percent': utilization_percent,
                'free_capacity_gb': (total_capacity - used_capacity) / 1024 / 1024 / 1024,
                'status': 'Success'
            })
            
            array.invalidate_cookie()
            logger.info(f"Successfully retrieved data from {array_name}")
            
        except Exception as e:
            logger.error(f"Error connecting to array {host}: {str(e)}")
            arrays_data.append({
                'array_name': f"Error - {host}",
                'host': host,
                'total_capacity_gb': 0,
                'used_capacity_gb': 0,
                'utilization_percent': 0,
                'free_capacity_gb': 0,
                'status': f'Error: {str(e)}'
            })
    
    return arrays_data

def create_capacity_chart(arrays_data, output_path):
    """
    Create a bar chart showing capacity utilization for multiple arrays
    
    Args:
        arrays_data (list): List of array capacity data dictionaries
        output_path (str): Path to save the chart image
    """
    # Filter out arrays with errors
    valid_arrays = [data for data in arrays_data if data['status'] == 'Success']
    
    if not valid_arrays:
        logger.warning("No valid array data to create chart")
        return False
    
    # Extract data for plotting
    array_names = [data['array_name'] for data in valid_arrays]
    used_capacity = [data['used_capacity_gb'] for data in valid_arrays]
    free_capacity = [data['free_capacity_gb'] for data in valid_arrays]
    utilization = [data['utilization_percent'] for data in valid_arrays]
    
    # Create figure with two subplots
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(max(14, len(array_names) * 1.5), 12))
    
    # Plot 1: Stacked bar chart for used vs free capacity
    x_pos = np.arange(len(array_names))
    bar_width = 0.6
    
    bars1 = ax1.bar(x_pos, used_capacity, bar_width, label='Used Capacity (GB)', color='#ff7f0e')
    bars2 = ax1.bar(x_pos, free_capacity, bar_width, bottom=used_capacity, label='Free Capacity (GB)', color='#1f77b4')
    
    ax1.set_xlabel('Storage Arrays')
    ax1.set_ylabel('Capacity (GB)')
    ax1.set_title('Pure Storage Array Capacity Utilization')
    ax1.set_xticks(x_pos)
    ax1.set_xticklabels(array_names, rotation=45, ha='right')
    ax1.legend()
    
    # Add value labels on bars
    for i, (used, free) in enumerate(zip(used_capacity, free_capacity)):
        total = used + free
        if total > 0:  # Avoid division by zero
            ax1.text(i, total + (total * 0.01), f'{used:.0f}GB\n({utilization[i]:.1f}%)', 
                    ha='center', va='bottom', fontsize=8)
    
    # Plot 2: Utilization percentage
    colors = ['green' if util < 70 else 'orange' if util < 85 else 'red' for util in utilization]
    bars = ax2.bar(x_pos, utilization, bar_width, color=colors, alpha=0.7)
    
    ax2.set_xlabel('Storage Arrays')
    ax2.set_ylabel('Utilization (%)')
    ax2.set_title('Array Utilization Percentage')
    ax2.set_xticks(x_pos)
    ax2.set_xticklabels(array_names, rotation=45, ha='right')
    ax2.axhline(y=70, color='orange', linestyle='--', alpha=0.7, label='70% Warning')
    ax2.axhline(y=85, color='red', linestyle='--', alpha=0.7, label='85% Critical')
    ax2.legend()
    
    # Add value labels on bars
    for i, util in enumerate(utilization):
        ax2.text(i, util + 1, f'{util:.1f}%', ha='center', va='bottom', fontsize=9)
    
    # Adjust layout and save
    plt.tight_layout()
    plt.savefig(output_path, dpi=300, bbox_inches='tight')
    plt.close()
    
    return True

def send_capacity_report_email(arrays_data, chart_path, recipient_emails, 
                             smtp_server='localhost', smtp_port=25, 
                             from_email='storage-monitor@yourcompany.com',
                             email_subject='Pure Storage Capacity Report'):
    """
    Send capacity report email with chart attachment
    
    Args:
        arrays_data (list): List of array capacity data
        chart_path (str): Path to the chart image
        recipient_emails (list): List of recipient email addresses
        smtp_server (str): SMTP server hostname
        smtp_port (int): SMTP server port
        from_email (str): Sender email address
        email_subject (str): Email subject
    """
    # Count arrays with errors
    error_count = sum(1 for data in arrays_data if data['status'] != 'Success')
    success_count = len(arrays_data) - error_count
    
    # Create email message
    msg = MIMEMultipart()
    msg['Subject'] = f'{email_subject} - {datetime.now().strftime("%Y-%m-%d %H:%M")}'
    msg['From'] = from_email
    msg['To'] = ', '.join(recipient_emails)
    
    # Create HTML content
    html_content = f"""
    <html>
    <head>
        <style>
            body {{ font-family: Arial, sans-serif; margin: 20px; }}
            h2 {{ color: #333; }}
            table {{ border-collapse: collapse; width: 100%; margin-bottom: 20px; }}
            th, td {{ border: 1px solid #ddd; padding: 10px; text-align: left; }}
            th {{ background-color: #f2f2f2; font-weight: bold; }}
            tr:nth-child(even) {{ background-color: #f9f9f9; }}
            .warning {{ background-color: #fff3cd; }}
            .critical {{ background-color: #f8d7da; }}
            .error {{ background-color: #f8d7da; }}
            .success {{ background-color: #d4edda; }}
            .summary {{ background-color: #e9ecef; padding: 15px; border-radius: 5px; margin-bottom: 20px; }}
        </style>
    </head>
    <body>
        <h2>Pure Storage Array Capacity Report</h2>
        
        <div class="summary">
            <p><strong>Report Summary:</strong></p>
            <p>Generated on: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}</p>
            <p>Arrays successfully queried: <strong>{success_count}</strong></p>
            <p>Arrays with errors: <strong>{error_count}</strong></p>
        </div>
        
        <table>
            <tr>
                <th>Array Name</th>
                <th>Host</th>
                <th>Total Capacity (GB)</th>
                <th>Used Capacity (GB)</th>
                <th>Free Capacity (GB)</th>
                <th>Utilization (%)</th>
                <th>Status</th>
            </tr>
    """
    
    for data in arrays_data:
        # Determine status and row class
        if data['status'] != 'Success':
            status = data['status']
            row_class = "error"
        else:
            utilization = data['utilization_percent']
            if utilization >= 85:
                status = "Critical"
                row_class = "critical"
            elif utilization >= 70:
                status = "Warning"
                row_class = "warning"
            else:
                status = "Normal"
                row_class = "success"
        
        html_content += f"""
            <tr class="{row_class}">
                <td>{data['array_name']}</td>
                <td>{data['host']}</td>
                <td>{data['total_capacity_gb']:,.0f}</td>
                <td>{data['used_capacity_gb']:,.0f}</td>
                <td>{data['free_capacity_gb']:,.0f}</td>
                <td>{data['utilization_percent']:.1f}%</td>
                <td>{status}</td>
            </tr>
        """
    
    html_content += """
        </table>
        <br>
    """
    
    # Only include chart if we have valid data
    if success_count > 0:
        html_content += '<img src="cid:capacity_chart" alt="Capacity Utilization Chart">'
    
    html_content += """
    </body>
    </html>
    """
    
    # Attach HTML content
    msg.attach(MIMEText(html_content, 'html'))
    
    # Attach chart image if it exists and we have valid data
    if success_count > 0 and os.path.exists(chart_path):
        with open(chart_path, 'rb') as f:
            img = MIMEImage(f.read())
            img.add_header('Content-ID', '<capacity_chart>')
            img.add_header('Content-Disposition', 'inline', filename='capacity_chart.png')
            msg.attach(img)
    
    # Send email
    try:
        logger.info(f"Sending email to {recipient_emails} via {smtp_server}:{smtp_port}")
        with smtplib.SMTP(smtp_server, smtp_port) as server:
            server.sendmail(from_email, recipient_emails, msg.as_string())
        logger.info(f"Email sent successfully to {recipient_emails}")
        return True
    except Exception as e:
        logger.error(f"Error sending email: {str(e)}")
        return False

def generate_and_send_capacity_report(array_hosts, usernames, passwords, recipient_emails, 
                                    smtp_server='localhost', smtp_port=25, 
                                    from_email='storage-monitor@yourcompany.com',
                                    email_subject='Pure Storage Capacity Report'):
    """
    Main function to generate and send capacity report for multiple arrays
    
    Args:
        array_hosts (list): List of array hostnames/IP addresses
        usernames (list): List of usernames for each array
        passwords (list): List of passwords for each array
        recipient_emails (list): List of recipient email addresses
        smtp_server (str): SMTP server hostname
        smtp_port (int): SMTP server port
        from_email (str): Sender email address
        email_subject (str): Email subject
    """
    if not array_hosts:
        logger.error("No arrays specified")
        return False
    
    logger.info(f"Processing {len(array_hosts)} arrays...")
    
    # Get capacity data from arrays
    arrays_data = get_array_capacity_utilization(array_hosts, usernames, passwords)
    
    # Create temporary file for chart
    with tempfile.NamedTemporaryFile(suffix='.png', delete=False) as temp_file:
        chart_path = temp_file.name
    
    try:
        # Create chart (only if we have valid data)
        chart_created = create_capacity_chart(arrays_data, chart_path)
        
        if chart_created:
            logger.info("Chart created successfully")
        else:
            logger.warning("Could not create chart due to lack of valid data")
        
        # Send email
        email_sent = send_capacity_report_email(
            arrays_data, 
            chart_path, 
            recipient_emails,
            smtp_server=smtp_server,
            smtp_port=smtp_port,
            from_email=from_email,
            email_subject=email_subject
        )
        
        return email_sent
        
    except Exception as e:
        logger.error(f"Error generating report: {str(e)}")
        return False
        
    finally:
        # Clean up temporary file
        if os.path.exists(chart_path):
            os.unlink(chart_path)

# Example usage and configuration
if __name__ == "__main__":
    # Configuration - replace with your actual values
    
    # Array connection details
    ARRAY_HOSTS = [
        "purearray1.company.com",
        "purearray2.company.com",
        "purearray3.company.com"
    ]
    
    # You can specify one username/password for all arrays, or individual ones for each
    ARRAY_USERNAMES = ["pureuser"]  # Single username for all arrays
    ARRAY_PASSWORDS = ["purepassword"]  # Single password for all arrays
    
    # Alternatively, specify individual credentials for each array:
    # ARRAY_USERNAMES = ["user1", "user2", "user3"]
    # ARRAY_PASSWORDS = ["pass1", "pass2", "pass3"]
    
    # Email configuration
    RECIPIENT_EMAILS = [
        "storage-team@yourcompany.com",
        "admin@yourcompany.com"
    ]
    
    SMTP_SERVER = "localhost"  # SMTP server is already configured and whitelisted
    SMTP_PORT = 25
    FROM_EMAIL = "storage-monitor@yourcompany.com"
    EMAIL_SUBJECT = "Pure Storage Capacity Report"
    
    # Generate and send report
    success = generate_and_send_capacity_report(
        ARRAY_HOSTS, 
        ARRAY_USERNAMES, 
        ARRAY_PASSWORDS, 
        RECIPIENT_EMAILS,
        smtp_server=SMTP_SERVER,
        smtp_port=SMTP_PORT,
        from_email=FROM_EMAIL,
        email_subject=EMAIL_SUBJECT
    )
    
    if success:
        logger.info("Capacity report process completed successfully")
    else:
        logger.error("Capacity report process failed")
```

## Key Features:

1. **Username/Password Authentication**: Uses Pure Storage username/password instead of API tokens
2. **Dynamic Array Handling**: Works with 1 to 30+ arrays automatically
3. **Flexible Credential Management**: Supports both shared and individual credentials for arrays
4. **Comprehensive Error Handling**: Gracefully handles connection errors and continues processing other arrays
5. **Detailed Reporting**: Includes both visual charts and tabular data in the email
6. **Status Indicators**: Color-codes arrays based on utilization thresholds and connection status
7. **Logging**: Provides detailed logging for troubleshooting

## Usage Instructions:

1. Install required packages:
```bash
pip install purestorage matplotlib numpy
```

2. Configure the script with your array details:
   - Update `ARRAY_HOSTS` with your Pure Storage array hostnames/IPs
   - Set `ARRAY_USERNAMES` and `ARRAY_PASSWORDS` (can use a single credential for all arrays or individual ones)
   - Configure email recipients in `RECIPIENT_EMAILS`
   - Adjust SMTP settings if different from default localhost configuration

3. Run the script:
```bash
python purestorage_capacity_report.py
```

## Customization Options:

- Adjust utilization thresholds (currently 70% warning, 85% critical)
- Modify chart appearance and colors
- Add more array metrics as needed
- Customize email template and content
- Add scheduling capabilities for regular reports

The script will automatically handle any number of arrays and provide both visual and tabular data in the email report. The SMTP configuration assumes the server is already set up and whitelisted as per your requirements.
