<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Particle Globe</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: radial-gradient(ellipse at center, #1a1a2e 0%, #0f0f23 70%, #000000 100%);
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            color: white;
        }

        #container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }

        #canvas {
            display: block;
            cursor: grab;
        }

        #canvas:active {
            cursor: grabbing;
        }

        .ui-controls {
            position: absolute;
            top: 50%;
            left: 50px;
            transform: translateY(-50%);
            z-index: 100;
        }

        .morph-button {
            padding: 15px 30px;
            background: rgba(255, 255, 255, 0.1);
            border: 2px solid #4ecdc4;
            color: #4ecdc4;
            font-size: 16px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            cursor: pointer;
            backdrop-filter: blur(10px);
            border-radius: 8px;
            transition: all 0.3s ease;
            margin-bottom: 20px;
            display: block;
            width: 200px;
            text-align: center;
        }

        .morph-button:hover {
            background: rgba(78, 205, 196, 0.2);
            box-shadow: 0 0 20px rgba(78, 205, 196, 0.5);
            transform: translateY(-2px);
        }

        .morph-button:active {
            transform: translateY(0);
        }

        .info {
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 10px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            max-width: 250px;
        }

        .info h3 {
            color: #4ecdc4;
            margin-bottom: 10px;
            font-size: 18px;
        }

        .info p {
            font-size: 14px;
            line-height: 1.4;
            opacity: 0.8;
            margin-bottom: 10px;
        }

        .controls-list {
            font-size: 12px;
            opacity: 0.6;
        }

        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #4ecdc4;
            font-size: 18px;
            z-index: 200;
        }

        .spinner {
            border: 3px solid rgba(78, 205, 196, 0.3);
            border-top: 3px solid #4ecdc4;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 20px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        @media (max-width: 768px) {
            .ui-controls {
                left: 20px;
                right: 20px;
                top: auto;
                bottom: 50px;
                transform: none;
            }
            
            .info {
                max-width: none;
            }
            
            .morph-button {
                width: 100%;
            }
        }
    </style>
</head>
<body>
    <div id="container">
        <div class="loading" id="loading">
            <div class="spinner"></div>
            Loading 3D Experience...
        </div>
        
        <canvas id="canvas"></canvas>
        
        <div class="ui-controls">
            <button class="morph-button" id="morphButton">Transform to Landscape</button>
            <button class="morph-button" id="resetButton">Return to Globe</button>
            
            <div class="info">
                <h3>Interactive 3D Scene</h3>
                <p>Drag to rotate the view and explore the particle system.</p>
                <div class="controls-list">
                    • Mouse: Rotate view<br>
                    • Scroll: Zoom in/out<br>
                    • Buttons: Morph particles
                </div>
            </div>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        class ParticleGlobe {
            constructor() {
                this.scene = new THREE.Scene();
                this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
                this.renderer = new THREE.WebGLRenderer({ 
                    canvas: document.getElementById('canvas'),
                    antialias: true,
                    alpha: true
                });
                
                this.particles = [];
                this.particleCount = 8000;
                this.globeRadius = 5;
                this.isTransitioning = false;
                this.currentShape = 'globe'; // 'globe' or 'landscape'
                
                this.mouse = { x: 0, y: 0 };
                this.mouseDown = false;
                this.rotation = { x: 0, y: 0 };
                this.targetRotation = { x: 0, y: 0 };
                
                this.init();
                this.createParticles();
                this.setupControls();
                this.animate();
                
                // Hide loading screen
                setTimeout(() => {
                    document.getElementById('loading').style.display = 'none';
                }, 1000);
            }
            
            init() {
                this.renderer.setSize(window.innerWidth, window.innerHeight);
                this.renderer.setClearColor(0x000000, 0);
                this.camera.position.z = 15;
                
                // Add ambient lighting
                const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
                this.scene.add(ambientLight);
                
                // Add point light
                const pointLight = new THREE.PointLight(0x4ecdc4, 1, 100);
                pointLight.position.set(10, 10, 10);
                this.scene.add(pointLight);
                
                // Add stars background
                this.createStarField();
            }
            
            createStarField() {
                const starGeometry = new THREE.BufferGeometry();
                const starCount = 1000;
                const positions = new Float32Array(starCount * 3);
                
                for (let i = 0; i < starCount * 3; i += 3) {
                    positions[i] = (Math.random() - 0.5) * 200;
                    positions[i + 1] = (Math.random() - 0.5) * 200;
                    positions[i + 2] = (Math.random() - 0.5) * 200;
                }
                
                starGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
                
                const starMaterial = new THREE.PointsMaterial({
                    color: 0xffffff,
                    size: 0.5,
                    transparent: true,
                    opacity: 0.6
                });
                
                const stars = new THREE.Points(starGeometry, starMaterial);
                this.scene.add(stars);
            }
            
            createParticles() {
                const geometry = new THREE.BufferGeometry();
                const positions = new Float32Array(this.particleCount * 3);
                const colors = new Float32Array(this.particleCount * 3);
                
                // Create globe positions
                for (let i = 0; i < this.particleCount; i++) {
                    const phi = Math.acos(-1 + (2 * i) / this.particleCount);
                    const theta = Math.sqrt(this.particleCount * Math.PI) * phi;
                    
                    const x = this.globeRadius * Math.cos(theta) * Math.sin(phi);
                    const y = this.globeRadius * Math.cos(phi);
                    const z = this.globeRadius * Math.sin(theta) * Math.sin(phi);
                    
                    positions[i * 3] = x;
                    positions[i * 3 + 1] = y;
                    positions[i * 3 + 2] = z;
                    
                    // Create gradient colors based on position
                    const hue = (y + this.globeRadius) / (this.globeRadius * 2);
                    const color = new THREE.Color().setHSL(0.5 + hue * 0.3, 0.8, 0.6);
                    colors[i * 3] = color.r;
                    colors[i * 3 + 1] = color.g;
                    colors[i * 3 + 2] = color.b;
                    
                    // Store original positions for morphing
                    this.particles.push({
                        globePos: { x, y, z },
                        landscapePos: this.generateLandscapePosition(i),
                        currentPos: { x, y, z }
                    });
                }
                
                geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
                geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
                
                const material = new THREE.PointsMaterial({
                    size: 0.05,
                    vertexColors: true,
                    transparent: true,
                    opacity: 0.8,
                    blending: THREE.AdditiveBlending
                });
                
                this.particleSystem = new THREE.Points(geometry, material);
                this.scene.add(this.particleSystem);
            }
            
            generateLandscapePosition(index) {
                // Create a landscape-like distribution
                const width = 12;
                const depth = 8;
                const x = (Math.random() - 0.5) * width;
                const z = (Math.random() - 0.5) * depth;
                
                // You can replace this with real elevation data
                // Example: y = this.getElevationFromData(x, z);
                
                // Current: Create hills and valleys using noise-like function
                const noise1 = Math.sin(x * 0.5) * Math.cos(z * 0.5) * 2;
                const noise2 = Math.sin(x * 1.2) * Math.cos(z * 1.2) * 0.5;
                const noise3 = Math.sin(x * 2.5) * Math.cos(z * 2.5) * 0.2;
                
                const y = noise1 + noise2 + noise3 - 2;
                
                return { x, y, z };
            }
            
            // Method to load real elevation data
            async loadElevationData(region = 'mount_everest') {
                const regions = {
                    mount_everest: {
                        // Everest region coordinates
                        bounds: { north: 28.1, south: 27.8, east: 87.0, west: 86.7 },
                        // You would load actual DEM data here
                        elevationFunction: (x, z) => {
                            // Simplified Everest-like peak
                            const distanceFromPeak = Math.sqrt(x*x + z*z);
                            return Math.max(0, 8.8 - distanceFromPeak * 2); // 8.8km high like Everest
                        }
                    },
                    grand_canyon: {
                        bounds: { north: 36.3, south: 35.9, east: -111.7, west: -112.3 },
                        elevationFunction: (x, z) => {
                            // Canyon-like depression
                            const canyonDepth = Math.sin(x * 0.3) * 2;
                            return -Math.abs(canyonDepth) - Math.abs(z * 0.5);
                        }
                    },
                    manhattan: {
                        bounds: { north: 40.8, south: 40.7, east: -73.9, west: -74.0 },
                        elevationFunction: (x, z) => {
                            // City skyline simulation
                            const buildingHeight = Math.random() * 0.4;
                            return buildingHeight + Math.sin(x * 5) * 0.1;
                        }
                    }
                };
                
                this.currentRegion = regions[region] || regions.mount_everest;
                this.regenerateLandscapeWithData();
            }
            
            regenerateLandscapeWithData() {
                // Regenerate landscape positions using loaded elevation data
                for (let i = 0; i < this.particleCount; i++) {
                    const width = 12;
                    const depth = 8;
                    const x = (Math.random() - 0.5) * width;
                    const z = (Math.random() - 0.5) * depth;
                    
                    // Use the elevation function from loaded region data
                    const y = this.currentRegion.elevationFunction(x, z);
                    
                    this.particles[i].landscapePos = { x, y, z };
                }
            }
            
            morphToLandscape() {
                if (this.isTransitioning) return;
                this.isTransitioning = true;
                this.currentShape = 'landscape';
                
                const duration = 2000; // 2 seconds
                const startTime = Date.now();
                
                const animate = () => {
                    const elapsed = Date.now() - startTime;
                    const progress = Math.min(elapsed / duration, 1);
                    const easeProgress = this.easeInOutCubic(progress);
                    
                    const positions = this.particleSystem.geometry.attributes.position.array;
                    
                    for (let i = 0; i < this.particleCount; i++) {
                        const particle = this.particles[i];
                        
                        // Interpolate between globe and landscape positions
                        const x = particle.globePos.x + (particle.landscapePos.x - particle.globePos.x) * easeProgress;
                        const y = particle.globePos.y + (particle.landscapePos.y - particle.globePos.y) * easeProgress;
                        const z = particle.globePos.z + (particle.landscapePos.z - particle.globePos.z) * easeProgress;
                        
                        positions[i * 3] = x;
                        positions[i * 3 + 1] = y;
                        positions[i * 3 + 2] = z;
                        
                        particle.currentPos = { x, y, z };
                    }
                    
                    this.particleSystem.geometry.attributes.position.needsUpdate = true;
                    
                    if (progress < 1) {
                        requestAnimationFrame(animate);
                    } else {
                        this.isTransitioning = false;
                    }
                };
                
                animate();
            }
            
            morphToGlobe() {
                if (this.isTransitioning) return;
                this.isTransitioning = true;
                this.currentShape = 'globe';
                
                const duration = 2000;
                const startTime = Date.now();
                
                const animate = () => {
                    const elapsed = Date.now() - startTime;
                    const progress = Math.min(elapsed / duration, 1);
                    const easeProgress = this.easeInOutCubic(progress);
                    
                    const positions = this.particleSystem.geometry.attributes.position.array;
                    
                    for (let i = 0; i < this.particleCount; i++) {
                        const particle = this.particles[i];
                        
                        // Interpolate from landscape back to globe
                        const x = particle.landscapePos.x + (particle.globePos.x - particle.landscapePos.x) * easeProgress;
                        const y = particle.landscapePos.y + (particle.globePos.y - particle.landscapePos.y) * easeProgress;
                        const z = particle.landscapePos.z + (particle.globePos.z - particle.landscapePos.z) * easeProgress;
                        
                        positions[i * 3] = x;
                        positions[i * 3 + 1] = y;
                        positions[i * 3 + 2] = z;
                        
                        particle.currentPos = { x, y, z };
                    }
                    
                    this.particleSystem.geometry.attributes.position.needsUpdate = true;
                    
                    if (progress < 1) {
                        requestAnimationFrame(animate);
                    } else {
                        this.isTransitioning = false;
                    }
                };
                
                animate();
            }
            
            easeInOutCubic(t) {
                return t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1;
            }
            
            setupControls() {
                const canvas = this.renderer.domElement;
                
                // Mouse controls
                canvas.addEventListener('mousedown', (e) => {
                    this.mouseDown = true;
                    this.mouse.x = e.clientX;
                    this.mouse.y = e.clientY;
                });
                
                canvas.addEventListener('mousemove', (e) => {
                    if (!this.mouseDown) return;
                    
                    const deltaX = e.clientX - this.mouse.x;
                    const deltaY = e.clientY - this.mouse.y;
                    
                    this.targetRotation.y += deltaX * 0.01;
                    this.targetRotation.x += deltaY * 0.01;
                    
                    this.mouse.x = e.clientX;
                    this.mouse.y = e.clientY;
                });
                
                canvas.addEventListener('mouseup', () => {
                    this.mouseDown = false;
                });
                
                // Zoom with scroll
                canvas.addEventListener('wheel', (e) => {
                    e.preventDefault();
                    this.camera.position.z += e.deltaY * 0.01;
                    this.camera.position.z = Math.max(5, Math.min(30, this.camera.position.z));
                });
                
                // Touch controls for mobile
                canvas.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    this.mouseDown = true;
                    this.mouse.x = e.touches[0].clientX;
                    this.mouse.y = e.touches[0].clientY;
                });
                
                canvas.addEventListener('touchmove', (e) => {
                    e.preventDefault();
                    if (!this.mouseDown) return;
                    
                    const deltaX = e.touches[0].clientX - this.mouse.x;
                    const deltaY = e.touches[0].clientY - this.mouse.y;
                    
                    this.targetRotation.y += deltaX * 0.01;
                    this.targetRotation.x += deltaY * 0.01;
                    
                    this.mouse.x = e.touches[0].clientX;
                    this.mouse.y = e.touches[0].clientY;
                });
                
                canvas.addEventListener('touchend', (e) => {
                    e.preventDefault();
                    this.mouseDown = false;
                });
                
                // Button controls
                document.getElementById('morphButton').addEventListener('click', () => {
                    this.morphToLandscape();
                });
                
                document.getElementById('resetButton').addEventListener('click', () => {
                    this.morphToGlobe();
                });
                
                // Window resize
                window.addEventListener('resize', () => {
                    this.camera.aspect = window.innerWidth / window.innerHeight;
                    this.camera.updateProjectionMatrix();
                    this.renderer.setSize(window.innerWidth, window.innerHeight);
                });
            }
            
            animate() {
                requestAnimationFrame(() => this.animate());
                
                // Smooth rotation interpolation
                this.rotation.x += (this.targetRotation.x - this.rotation.x) * 0.05;
                this.rotation.y += (this.targetRotation.y - this.rotation.y) * 0.05;
                
                // Apply rotation to particle system
                this.particleSystem.rotation.x = this.rotation.x;
                this.particleSystem.rotation.y = this.rotation.y;
                
                // Add subtle floating animation
                const time = Date.now() * 0.001;
                this.particleSystem.position.y = Math.sin(time * 0.5) * 0.1;
                
                // Animate particle colors
                const colors = this.particleSystem.geometry.attributes.color.array;
                for (let i = 0; i < this.particleCount; i++) {
                    const hueShift = Math.sin(time + i * 0.01) * 0.1;
                    const color = new THREE.Color().setHSL(0.5 + hueShift, 0.8, 0.6);
                    colors[i * 3] = color.r;
                    colors[i * 3 + 1] = color.g;
                    colors[i * 3 + 2] = color.b;
                }
                this.particleSystem.geometry.attributes.color.needsUpdate = true;
                
                this.renderer.render(this.scene, this.camera);
            }
        }
        
        // Initialize the app
        new ParticleGlobe();
    </script>
</body>
</html>