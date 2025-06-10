<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GPX Point Map</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            padding: 20px;
        }
        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 20px;
        }
        #map {
            height: 600px;
            width: 100%;
            border-radius: 8px;
            border: 2px solid #ddd;
        }
        .file-input {
            margin-bottom: 20px;
            text-align: center;
        }
        .file-input input[type="file"] {
            padding: 10px;
            border: 2px dashed #ccc;
            border-radius: 5px;
            background-color: #f9f9f9;
        }
        .stats {
            display: flex;
            justify-content: space-around;
            margin-top: 20px;
            padding: 15px;
            background-color: #f8f9fa;
            border-radius: 8px;
        }
        .stat-item {
            text-align: center;
        }
        .stat-value {
            font-size: 24px;
            font-weight: bold;
            color: #007bff;
        }
        .stat-label {
            font-size: 14px;
            color: #666;
        }
        .legend {
            position: absolute;
            bottom: 30px;
            left: 30px;
            background: white;
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            z-index: 1000;
        }
        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 5px;
        }
        .legend-color {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            margin-right: 8px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>GPX Point Map with Elevation</h1>
        
        <div class="file-input">
            <input type="file" id="gpxFile" accept=".gpx" />
            <p>Select your pittsburgh_2018.gpx file to visualize all GPS points with elevation data</p>
        </div>
        
        <div id="map"></div>
        
        <div class="stats" id="stats" style="display: none;">
            <div class="stat-item">
                <div class="stat-value" id="totalPoints">0</div>
                <div class="stat-label">Total Points</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="minElevation">0</div>
                <div class="stat-label">Min Elevation (m)</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="maxElevation">0</div>
                <div class="stat-label">Max Elevation (m)</div>
            </div>
            <div class="stat-item">
                <div class="stat-value" id="totalTracks">0</div>
                <div class="stat-label">Tracks</div>
            </div>
        </div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        // Initialize the map
        let map = L.map('map').setView([40.4406, -79.9959], 10); // Pittsburgh coordinates
        
        // Add tile layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);
        
        // Function to get color based on elevation
        function getElevationColor(elevation, minElev, maxElev) {
            if (!elevation) return '#666666'; // Gray for no elevation data
            
            const ratio = (elevation - minElev) / (maxElev - minElev);
            
            // Color gradient from blue (low) to red (high)
            if (ratio < 0.25) return '#0066ff'; // Blue
            else if (ratio < 0.5) return '#00ff66'; // Green
            else if (ratio < 0.75) return '#ffff00'; // Yellow
            else return '#ff0000'; // Red
        }
        
        // Function to parse GPX and add points to map
        function parseGPX(gpxText) {
            const parser = new DOMParser();
            const gpxDoc = parser.parseFromString(gpxText, 'text/xml');
            
            // Get all track points
            const trackPoints = gpxDoc.querySelectorAll('trkpt');
            const waypoints = gpxDoc.querySelectorAll('wpt');
            const routePoints = gpxDoc.querySelectorAll('rtept');
            
            let allPoints = [];
            let elevations = [];
            
            // Process track points
            trackPoints.forEach(point => {
                const lat = parseFloat(point.getAttribute('lat'));
                const lon = parseFloat(point.getAttribute('lon'));
                const elevElement = point.querySelector('ele');
                const elevation = elevElement ? parseFloat(elevElement.textContent) : null;
                
                allPoints.push({ lat, lon, elevation, type: 'track' });
                if (elevation !== null) elevations.push(elevation);
            });
            
            // Process waypoints
            waypoints.forEach(point => {
                const lat = parseFloat(point.getAttribute('lat'));
                const lon = parseFloat(point.getAttribute('lon'));
                const elevElement = point.querySelector('ele');
                const elevation = elevElement ? parseFloat(elevElement.textContent) : null;
                const nameElement = point.querySelector('name');
                const name = nameElement ? nameElement.textContent : 'Waypoint';
                
                allPoints.push({ lat, lon, elevation, type: 'waypoint', name });
                if (elevation !== null) elevations.push(elevation);
            });
            
            // Process route points
            routePoints.forEach(point => {
                const lat = parseFloat(point.getAttribute('lat'));
                const lon = parseFloat(point.getAttribute('lon'));
                const elevElement = point.querySelector('ele');
                const elevation = elevElement ? parseFloat(elevElement.textContent) : null;
                
                allPoints.push({ lat, lon, elevation, type: 'route' });
                if (elevation !== null) elevations.push(elevation);
            });
            
            if (allPoints.length === 0) {
                alert('No GPS points found in the GPX file!');
                return;
            }
            
            // Calculate elevation range
            const minElevation = Math.min(...elevations);
            const maxElevation = Math.max(...elevations);
            
            // Clear existing markers
            map.eachLayer(layer => {
                if (layer instanceof L.CircleMarker) {
                    map.removeLayer(layer);
                }
            });
            
            // Add points to map
            const bounds = L.latLngBounds();
            
            allPoints.forEach((point, index) => {
                const color = getElevationColor(point.elevation, minElevation, maxElevation);
                const radius = point.type === 'waypoint' ? 8 : 4;
                
                const marker = L.circleMarker([point.lat, point.lon], {
                    radius: radius,
                    fillColor: color,
                    color: '#000',
                    weight: 1,
                    opacity: 0.8,
                    fillOpacity: 0.8
                }).addTo(map);
                
                // Create popup content
                let popupContent = `
                    <strong>Point ${index + 1}</strong><br>
                    <strong>Type:</strong> ${point.type}<br>
                    <strong>Lat:</strong> ${point.lat.toFixed(6)}<br>
                    <strong>Lon:</strong> ${point.lon.toFixed(6)}<br>
                `;
                
                if (point.elevation !== null) {
                    popupContent += `<strong>Elevation:</strong> ${point.elevation.toFixed(1)}m<br>`;
                }
                
                if (point.name) {
                    popupContent += `<strong>Name:</strong> ${point.name}<br>`;
                }
                
                marker.bindPopup(popupContent);
                bounds.extend([point.lat, point.lon]);
            });
            
            // Fit map to show all points
            if (bounds.isValid()) {
                map.fitBounds(bounds, { padding: [10, 10] });
            }
            
            // Update statistics
            document.getElementById('totalPoints').textContent = allPoints.length;
            document.getElementById('minElevation').textContent = elevations.length > 0 ? minElevation.toFixed(1) : 'N/A';
            document.getElementById('maxElevation').textContent = elevations.length > 0 ? maxElevation.toFixed(1) : 'N/A';
            document.getElementById('totalTracks').textContent = gpxDoc.querySelectorAll('trk').length;
            document.getElementById('stats').style.display = 'flex';
            
            // Add legend
            addLegend(minElevation, maxElevation);
        }
        
        function addLegend(minElev, maxElev) {
            // Remove existing legend
            const existingLegend = document.querySelector('.legend');
            if (existingLegend) {
                existingLegend.remove();
            }
            
            const legend = document.createElement('div');
            legend.className = 'legend';
            legend.innerHTML = `
                <strong>Elevation Legend</strong><br>
                <div class="legend-item">
                    <div class="legend-color" style="background-color: #0066ff;"></div>
                    Low (${minElev.toFixed(0)}m)
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background-color: #00ff66;"></div>
                    Medium-Low
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background-color: #ffff00;"></div>
                    Medium-High
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background-color: #ff0000;"></div>
                    High (${maxElev.toFixed(0)}m)
                </div>
                <div class="legend-item">
                    <div class="legend-color" style="background-color: #666666;"></div>
                    No elevation data
                </div>
            `;
            
            document.body.appendChild(legend);
        }
        
        // File input handler
        document.getElementById('gpxFile').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    parseGPX(e.target.result);
                };
                reader.readAsText(file);
            }
        });
    </script>
</body>
</html>