I'll fix the grouping issue by using draw.io's grouping functionality. In draw.io, elements are grouped by setting the `parent` attribute to refer to the group container. Here's the corrected script:

```python
import xml.etree.ElementTree as ET
from xml.dom import minidom
import datetime

def create_drawio_diagram(arrays, hosts, connections, orientation='horizontal', 
                         array_h_spacing=300, host_h_spacing=250, 
                         array_v_spacing=200, host_v_spacing=150,
                         array_host_v_spacing=300, array_host_h_spacing=400,
                         show_legend=False):
    """
    Create a draw.io diagram showing Pure Storage arrays and ESX hosts relationships
    with FC and iSCSI connectivity, dual ports, and capacity information.
    
    Args:
        arrays: List of storage arrays
        hosts: List of ESX hosts
        connections: List of connections between arrays and hosts
        orientation: 'horizontal' or 'vertical' layout
        array_h_spacing: Horizontal spacing between arrays
        host_h_spacing: Horizontal spacing between hosts
        array_v_spacing: Vertical spacing between arrays (for vertical layout)
        host_v_spacing: Vertical spacing between hosts (for vertical layout)
        array_host_v_spacing: Vertical spacing between arrays and hosts (for horizontal layout)
        array_host_h_spacing: Horizontal spacing between arrays and hosts (for vertical layout)
        show_legend: Whether to show the legend (default: False)
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
        mxcell.set("style", f"shape=rectangle;whiteSpace=wrap;html=1;rounded=1;fillColor={utilization_color};strokeColor=#006BB3;fontColor=#000000;fontSize=12;")
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
        ports_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#BDBDBD;fontSize=11;")
        ports_cell.set("vertex", "1")
        ports_cell.set("parent", array_group_id)  # Parent is the group
        
        # Position ports container based on orientation
        if orientation == 'horizontal':
            ports_x = array_x - 10
            ports_y = array_y_pos + 145
            ports_width = 200
            ports_height = 20 + (len(array['ports']) * 25)
        else:
            ports_x = array_x + 185
            ports_y = array_y_pos - 10
            ports_width = 120
            ports_height = 20 + (len(array['ports']) * 25)
        
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
                port_y = ports_y + 20 + (j * 25)
            else:
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 25)
            
            port_cell = ET.SubElement(root, "mxCell")
            port_cell.set("id", port_id)
            port_cell.set("value", f"{port['type']}: {port['address']}")
            
            if port['type'] == 'FC':
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFE0B2;strokeColor=#F57C00;fontSize=10;"
            else:  # iSCSI
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#C8E6C9;strokeColor=#388E3C;fontSize=10;"
                
            port_cell.set("style", port_style)
            port_cell.set("vertex", "1")
            port_cell.set("parent", array_group_id)  # Parent is the group
            
            port_geo = ET.SubElement(port_cell, "mxGeometry")
            port_geo.set("x", str(port_x))
            port_geo.set("y", str(port_y))
            port_geo.set("width", str(ports_width - 20))
            port_geo.set("height", "20")
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
        mxcell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;rounded=1;fillColor=#E0E0E0;strokeColor=#616161;fontSize=12;")
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
        ports_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFFFFF;strokeColor=#BDBDBD;fontSize=11;")
        ports_cell.set("vertex", "1")
        ports_cell.set("parent", host_group_id)  # Parent is the group
        
        # Position ports container based on orientation
        if orientation == 'horizontal':
            ports_x = host_x - 10
            ports_y = host_y_pos - 20 - (len(host['ports']) * 25)
            ports_width = 170
            ports_height = 20 + (len(host['ports']) * 25)
        else:
            ports_x = host_x - 130
            ports_y = host_y_pos - 10
            ports_width = 120
            ports_height = 20 + (len(host['ports']) * 25)
        
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
                port_y = ports_y + 20 + (j * 25)
            else:
                port_x = ports_x + 10
                port_y = ports_y + 20 + (j * 25)
            
            port_cell = ET.SubElement(root, "mxCell")
            port_cell.set("id", port_id)
            port_cell.set("value", f"{port['type']}: {port['address']}")
            
            if port['type'] == 'FC':
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#FFE0B2;strokeColor=#F57C00;fontSize=10;"
            else:  # iSCSI
                port_style = "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#C8E6C9;strokeColor=#388E3C;fontSize=10;"
                
            port_cell.set("style", port_style)
            port_cell.set("vertex", "1")
            port_cell.set("parent", host_group_id)  # Parent is the group
            
            port_geo = ET.SubElement(port_cell, "mxGeometry")
            port_geo.set("x", str(port_x))
            port_geo.set("y", str(port_y))
            port_geo.set("width", str(ports_width - 20))
            port_geo.set("height", "20")
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
                line_style = "endArrow=classic;html=1;rounded=0;strokeWidth=2;strokeColor=#FF9800;dashed=0;"
            else:  # iSCSI
                line_style = "endArrow=classic;html=1;rounded=0;strokeWidth=2;strokeColor=#4CAF50;dashed=1;dashPattern=1 3;"
                
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
        legend_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#F5F5F5;strokeColor=#BDBDBD;fontSize=12;")
        legend_cell.set("vertex", "1")
        legend_cell.set("parent", "1")  # Legend is not part of any group
        
        legend_geo = ET.SubElement(legend_cell, "mxGeometry")
        if orientation == 'horizontal':
            legend_geo.set("x", str(diagram_width - 250))
            legend_geo.set("y", "50")
        else:
            legend_geo.set("x", str(diagram_width - 250))
            legend_geo.set("y", "50")
        legend_geo.set("width", "200")
        legend_geo.set("height", "150")
        legend_geo.set("as", "geometry")
    
    # Convert to XML string with pretty formatting
    rough_string = ET.tostring(mxfile, 'utf-8')
    reparsed = minidom.parseString(rough_string)
    pretty_xml = reparsed.toprettyxml(indent="  ")
    
    return pretty_xml

def main():
    # Sample data - replace with your actual storage and host information
    pure_arrays = [
        {
            'name': 'PURE-ARRAY-01',
            'model': 'FlashArray//X90',
            'version': '6.1.8',
            'total_capacity_gb': 102400,
            'used_capacity_gb': 81920,
            'ports': [
                {'type': 'FC', 'address': '50:0A:09:80:76:54:32:10'},
                {'type': 'FC', 'address': '50:0A:09:80:76:54:32:11'},
                {'type': 'iSCSI', 'address': 'iqn.2024-01.pure01:target01'},
                {'type': 'iSCSI', 'address': 'iqn.2024-01.pure01:target02'}
            ]
        },
        {
            'name': 'PURE-ARRAY-02', 
            'model': 'FlashArray//X70',
            'version': '6.2.3',
            'total_capacity_gb': 51200,
            'used_capacity_gb': 46080,
            'ports': [
                {'type': 'FC', 'address': '50:0B:09:81:76:54:32:20'},
                {'type': 'FC', 'address': '50:0B:09:81:76:54:32:21'},
                {'type': 'iSCSI', 'address': 'iqn.2024-01.pure02:target01'},
                {'type': 'iSCSI', 'address': 'iqn.2024-01.pure02:target02'}
            ]
        }
    ]
    
    esx_hosts = [
        {
            'name': 'ESX-HOST-01',
            'cluster': 'PROD-CLUSTER-01',
            'version': '7.0.3',
            'ports': [
                {'type': 'FC', 'address': '50:0A:09:80:11:11:11:11'},
                {'type': 'FC', 'address': '50:0A:09:80:11:11:11:12'}
            ]
        },
        {
            'name': 'ESX-HOST-02',
            'cluster': 'PROD-CLUSTER-01', 
            'version': '7.0.3',
            'ports': [
                {'type': 'FC', 'address': '50:0A:09:80:22:22:22:21'},
                {'type': 'FC', 'address': '50:0A:09:80:22:22:22:22'}
            ]
        },
        {
            'name': 'ESX-HOST-03',
            'cluster': 'DEV-CLUSTER-01',
            'version': '8.0.1',
            'ports': [
                {'type': 'iSCSI', 'address': 'iqn.2024-01.esx.host03:initiator01'},
                {'type': 'iSCSI', 'address': 'iqn.2024-01.esx.host03:initiator02'}
            ]
        }
    ]
    
    connections = [
        # FC connections
        {'array_port': '50:0A:09:80:76:54:32:10', 'host_port': '50:0A:09:80:11:11:11:11', 'protocol': 'FC'},
        {'array_port': '50:0A:09:80:76:54:32:11', 'host_port': '50:0A:09:80:11:11:11:12', 'protocol': 'FC'},
        {'array_port': '50:0A:09:80:76:54:32:10', 'host_port': '50:0A:09:80:22:22:22:21', 'protocol': 'FC'},
        {'array_port': '50:0A:09:80:76:54:32:11', 'host_port': '50:0A:09:80:22:22:22:22', 'protocol': 'FC'},
        
        {'array_port': '50:0B:09:81:76:54:32:20', 'host_port': '50:0A:09:80:11:11:11:11', 'protocol': 'FC'},
        {'array_port': '50:0B:09:81:76:54:32:21', 'host_port': '50:0A:09:80:22:22:22:22', 'protocol': 'FC'},
        
        # iSCSI connections
        {'array_port': 'iqn.2024-01.pure01:target01', 'host_port': 'iqn.2024-01.esx.host03:initiator01', 'protocol': 'iSCSI'},
        {'array_port': 'iqn.2024-01.pure01:target02', 'host_port': 'iqn.2024-01.esx.host03:initiator02', 'protocol': 'iSCSI'},
        {'array_port': 'iqn.2024-01.pure02:target01', 'host_port': 'iqn.2024-01.esx.host03:initiator01', 'protocol': 'iSCSI'},
        {'array_port': 'iqn.2024-01.pure02:target02', 'host_port': 'iqn.2024-01.esx.host03:initiator02', 'protocol': 'iSCSI'}
    ]
    
    # Configuration options
    orientation = 'horizontal'  # Change to 'vertical' for vertical layout
    show_legend = False  # Set to True to enable legend
    
    # Horizontal layout spacing
    array_h_spacing = 350  # Horizontal spacing between arrays
    host_h_spacing = 280   # Horizontal spacing between hosts
    array_host_v_spacing = 350  # Vertical spacing between arrays and hosts
    
    # Vertical layout spacing  
    array_v_spacing = 250  # Vertical spacing between arrays
    host_v_spacing = 200   # Vertical spacing between hosts
    array_host_h_spacing = 450  # Horizontal spacing between arrays and hosts
    
    # Generate the diagram XML
    xml_content = create_drawio_diagram(
        pure_arrays, 
        esx_hosts, 
        connections, 
        orientation=orientation,
        array_h_spacing=array_h_spacing,
        host_h_spacing=host_h_spacing,
        array_v_spacing=array_v_spacing,
        host_v_spacing=host_v_spacing,
        array_host_v_spacing=array_host_v_spacing,
        array_host_h_spacing=array_host_h_spacing,
        show_legend=show_legend
    )
    
    # Save to file
    filename = f"pure_storage_esx_connectivity_{orientation}.drawio.xml"
    with open(filename, 'w') as f:
        f.write(xml_content)
    
    print(f"Diagram saved as {filename}")
    print(f"Orientation: {orientation}")
    print(f"Legend enabled: {show_legend}")
    print("You can now import this file into draw.io Desktop v24.7.17")
    print("\nFeatures included:")
    print("- Proper grouping: Arrays/Hosts and their ports move together")
    print("- Each device has its own group container")
    print("- Port containers with individual port cells")
    print("- Legend is optional (disabled by default)")
    print("- All items are properly centered in the diagram")
    print("- Configurable spacing for both horizontal and vertical layouts")
    print("- Multiple Pure Storage arrays with capacity and utilization data")
    print("- FC and iSCSI connectivity with dual ports")
    print("- Color-coded connections (orange for FC, green for iSCSI)")
    print("- Utilization-based coloring (green <70%, orange 70-85%, red >85%)")

if __name__ == "__main__":
    main()
```

## Key Fixes for Grouping:

1. **Proper Group Containers**:
   - Each array and host now has a dedicated group container with `style="group"`
   - All elements (main device + ports) have their `parent` set to the group ID
   - Groups are properly structured with the correct hierarchy

2. **Group Structure**:
   - Group container (invisible, just for grouping)
   - Main device (array/host) as child of group
   - Port container as child of group  
   - Individual port cells as children of group

3. **Working Movement**:
   - When you select and move a device in draw.io, all its ports move with it
   - The group maintains the relative positioning of all elements

4. **Connections Outside Groups**:
   - Connection lines remain outside groups so they don't get moved accidentally
   - Connections reference the individual port cells by ID

The grouping should now work correctly in draw.io - when you move an array or host, all its associated port containers and individual port cells will move together as a single unit.
