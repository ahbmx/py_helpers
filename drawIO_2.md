I'll create a mockup dataframe with 30 arrays and 40 hosts with random connections between them. Here's the complete script:

```python
import xml.etree.ElementTree as ET
from xml.dom import minidom
import datetime
import random
import pandas as pd
import numpy as np

def generate_mock_data():
    """Generate mock data for 30 arrays and 40 hosts with random connections"""
    
    # Set random seed for reproducibility
    random.seed(42)
    np.random.seed(42)
    
    # Generate 30 Pure Storage arrays
    arrays = []
    array_models = ['FlashArray//X90', 'FlashArray//X70', 'FlashArray//X50', 'FlashArray//C60', 'FlashArray//C40']
    array_versions = ['6.1.8', '6.2.3', '6.0.5', '5.3.9', '6.3.1']
    
    for i in range(1, 31):
        model = random.choice(array_models)
        total_capacity = random.choice([51200, 102400, 204800, 409600, 819200])
        used_capacity = int(total_capacity * random.uniform(0.4, 0.95))
        
        array = {
            'name': f'PURE-ARRAY-{i:02d}',
            'model': model,
            'version': random.choice(array_versions),
            'total_capacity_gb': total_capacity,
            'used_capacity_gb': used_capacity,
            'ports': []
        }
        
        # Generate 2-4 ports per array (mix of FC and iSCSI)
        num_ports = random.randint(2, 4)
        fc_ports = random.randint(1, min(2, num_ports))
        iscsi_ports = num_ports - fc_ports
        
        # Generate FC ports
        for j in range(fc_ports):
            wwn = f"50:{random.randint(10,99):02d}:{random.randint(10,99):02d}:80:{random.randint(10,99):02d}:{random.randint(10,99):02d}:{random.randint(10,99):02d}:{random.randint(10,99):02d}"
            array['ports'].append({'type': 'FC', 'address': wwn})
        
        # Generate iSCSI ports
        for j in range(iscsi_ports):
            iqn = f"iqn.2024-01.pure{i:02d}:target{(j+1):02d}"
            array['ports'].append({'type': 'iSCSI', 'address': iqn})
        
        arrays.append(array)
    
    # Generate 40 ESX hosts
    hosts = []
    clusters = ['PROD-CLUSTER-01', 'PROD-CLUSTER-02', 'DEV-CLUSTER-01', 'DEV-CLUSTER-02', 'TEST-CLUSTER-01']
    host_versions = ['7.0.3', '8.0.1', '7.0.2', '8.0.2', '6.7.0']
    
    for i in range(1, 41):
        host = {
            'name': f'ESX-HOST-{i:02d}',
            'cluster': random.choice(clusters),
            'version': random.choice(host_versions),
            'ports': []
        }
        
        # Generate 2-4 ports per host (mix of FC and iSCSI based on cluster)
        num_ports = random.randint(2, 4)
        
        # Production clusters mostly use FC, dev/test use more iSCSI
        if host['cluster'].startswith('PROD'):
            fc_ports = min(num_ports, random.randint(2, 4))
            iscsi_ports = num_ports - fc_ports
        else:
            iscsi_ports = min(num_ports, random.randint(2, 4))
            fc_ports = num_ports - iscsi_ports
        
        # Generate FC ports
        for j in range(fc_ports):
            wwn = f"50:{random.randint(10,99):02d}:{random.randint(10,99):02d}:80:{i:02d}{random.randint(10,99):02d}:{random.randint(10,99):02d}:{random.randint(10,99):02d}:{random.randint(10,99):02d}"
            host['ports'].append({'type': 'FC', 'address': wwn})
        
        # Generate iSCSI ports
        for j in range(iscsi_ports):
            iqn = f"iqn.2024-01.esx.host{i:02d}:initiator{(j+1):02d}"
            host['ports'].append({'type': 'iSCSI', 'address': iqn})
        
        hosts.append(host)
    
    # Generate random connections between arrays and hosts
    connections = []
    
    # Create a pool of all available ports
    all_array_ports = []
    all_host_ports = []
    
    for array in arrays:
        for port in array['ports']:
            all_array_ports.append({
                'array_name': array['name'],
                'port_type': port['type'],
                'port_address': port['address']
            })
    
    for host in hosts:
        for port in host['ports']:
            all_host_ports.append({
                'host_name': host['name'],
                'port_type': port['type'],
                'port_address': port['address']
            })
    
    # Create connections - ensure each host has at least 1 connection
    connected_hosts = set()
    
    # First pass: connect each host to at least one array
    for host_port in all_host_ports:
        if host_port['host_name'] not in connected_hosts:
            # Find compatible array port
            compatible_ports = [ap for ap in all_array_ports if ap['port_type'] == host_port['port_type']]
            if compatible_ports:
                array_port = random.choice(compatible_ports)
                connections.append({
                    'array_port': array_port['port_address'],
                    'host_port': host_port['port_address'],
                    'protocol': array_port['port_type']
                })
                connected_hosts.add(host_port['host_name'])
    
    # Second pass: add additional random connections (about 2-4 per host on average)
    total_connections = len(hosts) * random.randint(2, 4)
    while len(connections) < total_connections and len(all_array_ports) > 0 and len(all_host_ports) > 0:
        array_port = random.choice(all_array_ports)
        host_port = random.choice([hp for hp in all_host_ports if hp['port_type'] == array_port['port_type']])
        
        connections.append({
            'array_port': array_port['port_address'],
            'host_port': host_port['port_address'],
            'protocol': array_port['port_type']
        })
    
    # Remove duplicates
    unique_connections = []
    seen = set()
    for conn in connections:
        key = (conn['array_port'], conn['host_port'])
        if key not in seen:
            unique_connections.append(conn)
            seen.add(key)
    
    return arrays, hosts, unique_connections

def create_drawio_diagram(arrays, hosts, connections, orientation='horizontal', 
                         array_h_spacing=300, host_h_spacing=250, 
                         array_v_spacing=200, host_v_spacing=150,
                         array_host_v_spacing=300, array_host_h_spacing=400,
                         show_legend=False):
    """
    Create a draw.io diagram showing Pure Storage arrays and ESX hosts relationships
    with FC and iSCSI connectivity, dual ports, and capacity information.
    """
    
    # Create the root mxfile element
    mxfile = ET.Element("mxfile")
    mxfile.set("host", "app.diagrams.net")
    mxfile.set("modified", datetime.datetime.now().isoformat() + "Z")
    mxfile.set("agent", "Python Script")
    mxfile.set("version", "24.7.17")
    mxfile.set("type", "device")
    
    # Create diagram element
    diagram = ET.SubElement(mxfile, "diagram")
    diagram.set("name", "Pure Storage - ESX Host Connectivity")
    diagram.set("id", "pure-esx-connectivity")
    
    # Calculate dimensions and centering based on number of nodes
    num_arrays = len(arrays)
    num_hosts = len(hosts)
    
    if orientation == 'horizontal':
        # Horizontal layout: Arrays on top, hosts on bottom
        diagram_width = max(num_arrays * array_h_spacing, num_hosts * host_h_spacing) + 400
        diagram_height = array_host_v_spacing + 400
        
        # Center arrays horizontally
        total_arrays_width = num_arrays * array_h_spacing
        array_x_start = (diagram_width - total_arrays_width) / 2 + array_h_spacing / 2
        array_y = 100
        
        # Center hosts horizontally
        total_hosts_width = num_hosts * host_h_spacing
        host_x_start = (diagram_width - total_hosts_width) / 2 + host_h_spacing / 2
        host_y = array_y + array_host_v_spacing
        
    else:
        # Vertical layout: Arrays on left, hosts on right
        diagram_width = array_host_h_spacing + 400
        diagram_height = max(num_arrays * array_v_spacing, num_hosts * host_v_spacing) + 300
        
        # Center arrays vertically
        total_arrays_height = num_arrays * array_v_spacing
        array_y_start = (diagram_height - total_arrays_height) / 2 + array_v_spacing / 2
        array_x = 100
        
        # Center hosts vertically
        total_hosts_height = num_hosts * host_v_spacing
        host_y_start = (diagram_height - total_hosts_height) / 2 + host_v_spacing / 2
        host_x = array_x + array_host_h_spacing
    
    # Create mxGraphModel
    mxgraphmodel = ET.SubElement(diagram, "mxGraphModel")
    mxgraphmodel.set("dx", str(diagram_width))
    mxgraphmodel.set("dy", str(diagram_height))
    mxgraphmodel.set("grid", "1")
    mxgraphmodel.set("gridSize", "10")
    mxgraphmodel.set("guides", "1")
    mxgraphmodel.set("tooltips", "1")
    mxgraphmodel.set("connect", "1")
    mxgraphmodel.set("arrows", "1")
    mxgraphmodel.set("fold", "1")
    mxgraphmodel.set("page", "1")
    mxgraphmodel.set("pageScale", "1")
    mxgraphmodel.set("pageWidth", "827")
    mxgraphmodel.set("pageHeight", "1169")
    mxgraphmodel.set("math", "0")
    mxgraphmodel.set("shadow", "0")
    
    # Create root element
    root = ET.SubElement(mxgraphmodel, "root")
    
    # Add mxCell for root
    mxcell0 = ET.SubElement(root, "mxCell")
    mxcell0.set("id", "0")
    
    mxcell1 = ET.SubElement(root, "mxCell")
    mxcell1.set("id", "1")
    mxcell1.set("parent", "0")
    
    # Create array shapes
    array_shapes = {}
    array_groups = {}
    
    for i, array in enumerate(arrays):
        array_group_id = f"array_group_{i+1}"
        array_id = f"array_{i+1}"
        
        # Create group for array and its ports
        group_cell = ET.SubElement(root, "mxCell")
        group_cell.set("id", array_group_id)
        group_cell.set("value", "")
        group_cell.set("style", "group")
        group_cell.set("vertex", "1")
        group_cell.set("connectable", "0")
        group_cell.set("parent", "1")
        
        group_geo = ET.SubElement(group_cell, "mxGeometry")
        group_geo.set("x", "0")
        group_geo.set("y", "0")
        group_geo.set("width", "1")
        group_geo.set("height", "1")
        group_geo.set("as", "geometry")
        
        # Calculate utilization percentage and color
        utilization_pct = (array['used_capacity_gb'] / array['total_capacity_gb']) * 100
        if utilization_pct > 85:
            utilization_color = "#FF5252"  # Red for high utilization
        elif utilization_pct > 70:
            utilization_color = "#FFB74D"  # Orange for medium utilization
        else:
            utilization_color = "#66BB6A"  # Green for low utilization
        
        # Set position based on orientation
        if orientation == 'horizontal':
            array_x = array_x_start + (i * array_h_spacing) - 90  # Center the array
            array_y_pos = array_y
        else:
            array_x_pos = array_x
            array_y_pos = array_y_start + (i * array_v_spacing) - 70  # Center the array
        
        # Array shape (child of group)
        mxcell = ET.SubElement(root, "mxCell")
        mxcell.set("id", array_id)
        mxcell.set("value", f"{array['name']}\n{array['model']}\n{array['version']}\n\nCapacity: {array['total_capacity_gb']:,.0f} GB\nUsed: {array['used_capacity_gb']:,.0f} GB ({utilization_pct:.1f}%)")
        mxcell.set("style", f"shape=rectangle;whiteSpace=wrap;html=1;rounded=1;fillColor={utilization_color};strokeColor=#006BB3;fontColor=#000000;fontSize=10;")
        mxcell.set("vertex", "1")
        mxcell.set("parent", array_group_id)  # Parent is the group
        
        geometry = ET.SubElement(mxcell, "mxGeometry")
        geometry.set("x", str(array_x))
        geometry.set("y", str(array_y_pos))
        geometry.set("width", "180")
        geometry.set("height", "140")
        geometry.set("as", "geometry")
        
        # Create port container for array (child of group)
        ports_container_id = f"{array_id}_ports"
        ports_cell = ET.SubElement(root, "mxCell")
        ports_cell.set("id", ports_container_id)
        ports_cell.set("value", "Ports")
        ports_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#BDBDBD;fontSize=9;")
        ports_cell.set("vertex", "1")
        ports_cell.set("parent", array_group_id)  # Parent is the group
        
        # Position ports container based on orientation
        if orientation == 'horizontal':
            ports_x = array_x - 10
            ports_y = array_y_pos + 145
            ports_width = 200
            ports_height = 20 + (len(array['ports']) * 20)
        else:
            ports_x = array_x + 185
            ports_y = array_y_pos - 10
            ports_width = 120
            ports_height = 20 + (len(array['ports']) * 20)
        
        ports_geo = ET.SubElement(ports_cell, "mxGeometry")
        ports_geo.set("x", str(ports_x))
        ports_geo.set("y", str(ports_y))
        ports_geo.set("width", str(ports_width))
        ports_geo.set("height", str(ports_height))
        ports_geo.set("as", "geometry")
        
        # Add individual port cells inside the container (children of group)
        for j, port in enumerate(array['ports']):
            port_id = f"{array_id}_port_{j+1}"
            
            # Set port position inside container
            if orientation == 'horizontal':
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 20)
            else:
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 20)
            
            port_cell = ET.SubElement(root, "mxCell")
            port_cell.set("id", port_id)
            port_cell.set("value", f"{port['type']}: {port['address']}")
            
            if port['type'] == 'FC':
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFE0B2;strokeColor=#F57C00;fontSize=8;"
            else:  # iSCSI
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#C8E6C9;strokeColor=#388E3C;fontSize=8;"
                
            port_cell.set("style", port_style)
            port_cell.set("vertex", "1")
            port_cell.set("parent", array_group_id)  # Parent is the group
            
            port_geo = ET.SubElement(port_cell, "mxGeometry")
            port_geo.set("x", str(port_x))
            port_geo.set("y", str(port_y))
            port_geo.set("width", str(ports_width - 20))
            port_geo.set("height", "18")
            port_geo.set("as", "geometry")
            
            array_shapes[port['address']] = port_id
        
        array_shapes[array['name']] = array_id
        array_groups[array['name']] = array_group_id
    
    # Create ESX host shapes
    host_shapes = {}
    host_groups = {}
    
    for i, host in enumerate(hosts):
        host_group_id = f"host_group_{i+1}"
        host_id = f"host_{i+1}"
        
        # Create group for host and its ports
        group_cell = ET.SubElement(root, "mxCell")
        group_cell.set("id", host_group_id)
        group_cell.set("value", "")
        group_cell.set("style", "group")
        group_cell.set("vertex", "1")
        group_cell.set("connectable", "0")
        group_cell.set("parent", "1")
        
        group_geo = ET.SubElement(group_cell, "mxGeometry")
        group_geo.set("x", "0")
        group_geo.set("y", "0")
        group_geo.set("width", "1")
        group_geo.set("height", "1")
        group_geo.set("as", "geometry")
        
        # Set position based on orientation
        if orientation == 'horizontal':
            host_x = host_x_start + (i * host_h_spacing) - 75  # Center the host
            host_y_pos = host_y
        else:
            host_x_pos = host_x
            host_y_pos = host_y_start + (i * host_v_spacing) - 40  # Center the host
        
        # ESX host shape (child of group)
        mxcell = ET.SubElement(root, "mxCell")
        mxcell.set("id", host_id)
        mxcell.set("value", f"{host['name']}\n{host['cluster']}\nvSphere {host['version']}")
        mxcell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;rounded=1;fillColor=#E0E0E0;strokeColor=#616161;fontSize=10;")
        mxcell.set("vertex", "1")
        mxcell.set("parent", host_group_id)  # Parent is the group
        
        geometry = ET.SubElement(mxcell, "mxGeometry")
        geometry.set("x", str(host_x))
        geometry.set("y", str(host_y_pos))
        geometry.set("width", "150")
        geometry.set("height", "80")
        geometry.set("as", "geometry")
        
        # Create port container for host (child of group)
        ports_container_id = f"{host_id}_ports"
        ports_cell = ET.SubElement(root, "mxCell")
        ports_cell.set("id", ports_container_id)
        ports_cell.set("value", "Ports")
        ports_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#BDBDBD;fontSize=9;")
        ports_cell.set("vertex", "1")
        ports_cell.set("parent", host_group_id)  # Parent is the group
        
        # Position ports container based on orientation
        if orientation == 'horizontal':
            ports_x = host_x - 10
            ports_y = host_y_pos - 20 - (len(host['ports']) * 20)
            ports_width = 170
            ports_height = 20 + (len(host['ports']) * 20)
        else:
            ports_x = host_x - 130
            ports_y = host_y_pos - 10
            ports_width = 120
            ports_height = 20 + (len(host['ports']) * 20)
        
        ports_geo = ET.SubElement(ports_cell, "mxGeometry")
        ports_geo.set("x", str(ports_x))
        ports_geo.set("y", str(ports_y))
        ports_geo.set("width", str(ports_width))
        ports_geo.set("height", str(ports_height))
        ports_geo.set("as", "geometry")
        
        # Add individual port cells inside the container (children of group)
        for j, port in enumerate(host['ports']):
            port_id = f"{host_id}_port_{j+1}"
            
            # Set port position inside container
            if orientation == 'horizontal':
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 20)
            else:
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 20)
            
            port_cell = ET.SubElement(root, "mxCell")
            port_cell.set("id", port_id)
            port_cell.set("value", f"{port['type']}: {port['address']}")
            
            if port['type'] == 'FC':
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFE0B2;strokeColor=#F57C00;fontSize=8;"
            else:  # iSCSI
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#C8E6C9;strokeColor=#388E3C;fontSize=8;"
                
            port_cell.set("style", port_style)
            port_cell.set("vertex", "1")
            port_cell.set("parent", host_group_id)  # Parent is the group
            
            port_geo = ET.SubElement(port_cell, "mxGeometry")
            port_geo.set("x", str(port_x))
            port_geo.set("y", str(port_y))
            port_geo.set("width", str(ports_width - 20))
            port_geo.set("height", "18")
            port_geo.set("as", "geometry")
            
            host_shapes[port['address']] = port_id
        
        host_shapes[host['name']] = host_id
        host_groups[host['name']] = host_group_id
    
    # Create connections
    connection_id = 1000
    
    for connection in connections:
        source_id = array_shapes.get(connection['array_port'])
        target_id = host_shapes.get(connection['host_port'])
        
        if source_id and target_id:
            # Connection line
            mxcell = ET.SubElement(root, "mxCell")
            mxcell.set("id", f"conn_{connection_id}")
            mxcell.set("value", f"{connection['protocol']}")
            
            if connection['protocol'] == 'FC':
                line_style = "endArrow=classic;html=1;rounded=0;strokeWidth=1;strokeColor=#FF9800;dashed=0;"
            else:  # iSCSI
                line_style = "endArrow=classic;html=1;rounded=0;strokeWidth=1;strokeColor=#4CAF50;dashed=1;dashPattern=1 3;"
                
            mxcell.set("style", line_style)
            mxcell.set("edge", "1")
            mxcell.set("parent", "1")  # Connections are not part of groups
            mxcell.set("source", source_id)
            mxcell.set("target", target_id)
            
            geometry = ET.SubElement(mxcell, "mxGeometry")
            geometry.set("relative", "1")
            geometry.set("as", "geometry")
            
            connection_id += 1
    
    # Add legend if enabled
    if show_legend:
        legend_id = "legend"
        legend_cell = ET.SubElement(root, "mxCell")
        legend_cell.set("id", legend_id)
        legend_cell.set("value", "Legend:\n Storage Utilization < 70%\n Storage Utilization 70-85%\n Storage Utilization > 85%\n\n FC Connections\n iSCSI Connections")
        legend_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#F5F5F5;strokeColor=#BDBDBD;fontSize=10;")
        legend_cell.set("vertex", "1")
        legend_cell.set("parent", "1")  # Legend is not part of any group
        
        legend_geo = ET.SubElement(legend_cell, "mxGeometry")
        if orientation == 'horizontal':
            legend_geo.set("x", str(diagram_width - 200))
            legend_geo.set("y", "50")
        else:
            legend_geo.set("x", str(diagram_width - 200))
            legend_geo.set("y", "50")
        legend_geo.set("width", "180")
        legend_geo.set("height", "130")
        legend_geo.set("as", "geometry")
    
    # Convert to XML string with pretty formatting
    rough_string = ET.tostring(mxfile, 'utf-8')
    reparsed = minidom.parseString(rough_string)
    pretty_xml = reparsed.toprettyxml(indent="  ")
    
    return pretty_xml

def main():
    # Generate mock data
    print("Generating mock data with 30 arrays and 40 hosts...")
    arrays, hosts, connections = generate_mock_data()
    
    # Create summary DataFrame
    array_df = pd.DataFrame(arrays)
    host_df = pd.DataFrame(hosts)
    
    print(f"Generated {len(arrays)} arrays:")
    print(f"  - Models: {array_df['model'].value_counts().to_dict()}")
    print(f"  - Capacity range: {array_df['total_capacity_gb'].min():,} - {array_df['total_capacity_gb'].max():,} GB")
    
    print(f"\nGenerated {len(hosts)} hosts:")
    print(f"  - Clusters: {host_df['cluster'].value_counts().to_dict()}")
    
    print(f"\nGenerated {len(connections)} connections:")
    protocol_counts = pd.Series([conn['protocol'] for conn in connections]).value_counts()
    print(f"  - Protocols: {protocol_counts.to_dict()}")
    
    # Configuration options
    orientation = 'horizontal'  # Use 'vertical' for vertical layout
    show_legend = True
    
    # Adjust spacing for larger dataset
    if orientation == 'horizontal':
        array_h_spacing = 250  # Reduced spacing for more arrays
        host_h_spacing = 200   # Reduced spacing for more hosts
        array_host_v_spacing = 400  # More vertical space for connections
    else:
        array_v_spacing = 180  # Reduced vertical spacing
        host_v_spacing = 150   # Reduced vertical spacing
        array_host_h_spacing = 400  # Horizontal spacing
    
    # Generate the diagram XML
    print("\nGenerating diagram...")
    xml_content = create_drawio_diagram(
        arrays, 
        hosts, 
        connections, 
        orientation=orientation,
        array_h_spacing=array_h_spacing,
        host_h_spacing=host_h_spacing,
        array_v_spacing=200,
        host_v_spacing=150,
        array_host_v_spacing=array_host_v_spacing,
        array_host_h_spacing=400,
        show_legend=show_legend
    )
    
    # Save to file
    filename = f"pure_storage_esx_connectivity_large_{orientation}.drawio.xml"
    with open(filename, 'w') as f:
        f.write(xml_content)
    
    print(f"Diagram saved as {filename}")
    print(f"Orientation: {orientation}")
    print(f"Legend enabled: {show_legend}")
    print("\nDiagram features:")
    print("- 30 Pure Storage arrays with random models, capacities, and utilization")
    print("- 40 ESX hosts across multiple clusters with different vSphere versions")
    print("- Realistic port configurations (FC and iSCSI)")
    print("- Random connections between arrays and hosts")
    print("- Proper grouping: Arrays/Hosts and their ports move together")
    print("- Color-coded utilization and connections")
    print("- Centered layout with optimized spacing")

if __name__ == "__main__":
    main()
```

## Key Features of the Mock Data Generator:

1. **30 Pure Storage Arrays**:
   - Random models (X90, X70, X50, C60, C40)
   - Random capacities (51TB to 819TB)
   - Realistic utilization levels (40-95%)
   - Mixed FC and iSCSI ports (2-4 ports per array)

2. **40 ESX Hosts**:
   - Across 5 different clusters (PROD, DEV, TEST)
   - Different vSphere versions
   - Production clusters favor FC, dev/test favor iSCSI
   - 2-4 ports per host

3. **Realistic Connections**:
   - Each host has at least one connection
   - Additional random connections (2-4 per host on average)
   - Protocol-compatible connections only (FC-FC, iSCSI-iSCSI)

4. **Optimized Layout**:
   - Reduced spacing to accommodate more nodes
   - Smaller font sizes for better fit
   - Thinner connection lines to reduce visual clutter

## Usage:

The script will generate a large-scale diagram with:
- 30 storage arrays with realistic specifications
- 40 ESX hosts across multiple environments  
- 100+ connections between them
- All properly grouped and formatted

You can adjust the `orientation` variable to switch between horizontal and vertical layouts based on your preference. The diagram will be saved as `pure_storage_esx_connectivity_large_{orientation}.drawio.xml`.
