<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>SOLAR // TACTICAL 3D SIMULATOR</title>

  <!-- Load Google Fonts: Inter & JetBrains Mono -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet" />

  <!-- Load Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
        },
      },
    };
  </script>

  <!-- Load Three.js r128 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

  <style>
    /* Strict High-Tech Styling using #000000, #ff0000, and #ffffff */
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    html, body {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      background-color: #000000;
      color: #ffffff;
      overflow: hidden;
      user-select: none;
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
      touch-action: none;
    }

    /* Subtle High-Tech Scanline Grid Overlay */
    .hud-scanlines {
      background-image: linear-gradient(
        rgba(255, 255, 255, 0.025) 1px,
        transparent 1px
      );
      background-size: 100% 4px;
      pointer-events: none;
    }

    /* Custom Scrollbar for Inspector Drawer */
    ::-webkit-scrollbar {
      width: 5px;
      height: 5px;
    }
    ::-webkit-scrollbar-track {
      background: #000000;
    }
    ::-webkit-scrollbar-thumb {
      background: #ff0000;
      border-radius: 9999px;
    }

    .no-scrollbar::-webkit-scrollbar {
      display: none;
    }
    .no-scrollbar {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }

    /* High-Tech Range Slider */
    input[type="range"].cyber-slider {
      -webkit-appearance: none;
      appearance: none;
      background: rgba(255, 255, 255, 0.2);
      height: 4px;
      border-radius: 999px;
      outline: none;
    }
    input[type="range"].cyber-slider::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: #ff0000;
      border: 2px solid #ffffff;
      cursor: pointer;
      box-shadow: 0 0 10px #ff0000;
    }
  </style>
</head>
<body class="bg-[#000000] text-[#ffffff] font-sans antialiased relative">

  <!-- Full-Viewport 3D WebGL Canvas Container -->
  <div id="canvasWrapper" class="fixed inset-0 z-0 bg-[#000000]">
    <canvas id="solarCanvas"></canvas>
  </div>

  <!-- Subtle CRT/HUD Scanline Layer -->
  <div class="fixed inset-0 z-10 hud-scanlines"></div>

  <!-- Floating Hover Reticle Tooltip (Desktop) -->
  <div
    id="hoverTooltip"
    class="fixed z-30 pointer-events-none hidden px-3 py-1.5 rounded border border-[#ff0000] bg-[#000000]/92 text-[#ffffff] font-mono text-xs shadow-[0_0_15px_rgba(255,0,0,0.65)]"
  >
    <div class="flex items-center gap-2">
      <span class="w-2 h-2 rounded-full bg-[#ff0000] animate-ping"></span>
      <span id="hoverTooltipText" class="font-bold tracking-wider uppercase">SATURN</span>
    </div>
    <div class="text-[10px] text-[#ff0000] mt-0.5">[CLICK TO LOCK &amp; INSPECT]</div>
  </div>

  <!-- TOP HUD HEADER BAR -->
  <header class="fixed top-0 left-0 right-0 z-20 p-3 sm:p-4 pointer-events-none">
    <div class="max-w-[1600px] mx-auto flex flex-wrap items-center justify-between gap-3">
      <!-- Brand / System Title -->
      <div class="pointer-events-auto flex items-center gap-3 px-3.5 py-2 rounded-xl border border-[#ff0000]/75 bg-[#000000]/88 backdrop-blur-md shadow-[0_0_25px_rgba(255,0,0,0.28)]">
        <div class="w-8 h-8 rounded-lg border border-[#ff0000] bg-[#ff0000]/15 flex items-center justify-center shrink-0">
          <svg class="w-4 h-4 text-[#ff0000]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
            <circle cx="12" cy="12" r="4" />
            <ellipse cx="12" cy="12" rx="10" ry="4" transform="rotate(-25 12 12)" />
          </svg>
        </div>
        <div>
          <div class="flex items-center gap-2">
            <h1 class="text-sm sm:text-base font-extrabold tracking-widest uppercase text-[#ffffff]">
              SOLAR<span class="text-[#ff0000]">//</span>SIMULATOR
            </h1>
            <span class="hidden md:inline-block text-[10px] font-mono px-1.5 py-0.5 rounded border border-[#ff0000] text-[#ff0000]">
              33 BODIES // RIGID LOCK
            </span>
          </div>
          <p class="text-[10px] sm:text-[11px] font-mono text-[#ffffff]/65">
            CLICK ANY PLANET OR MOON &bull; DRAG TO ORBIT &bull; SCROLL TO ZOOM
          </p>
        </div>
      </div>

      <!-- Top Right Simulation Controls -->
      <div class="pointer-events-auto flex flex-wrap items-center gap-2 bg-[#000000]/88 backdrop-blur-md p-2 rounded-xl border border-[#ffffff]/20">
        <!-- Orbit Speed Control -->
        <div class="flex items-center gap-2 px-2.5 py-1 rounded-lg border border-[#ffffff]/15">
          <span class="text-[10px] font-mono uppercase text-[#ffffff]/70">SPEED</span>
          <input
            id="speedSlider"
            type="range"
            min="0"
            max="5"
            step="0.25"
            value="1"
            class="cyber-slider w-16 sm:w-20"
            aria-label="Orbital Velocity"
          />
          <span id="speedValueLabel" class="text-[11px] font-mono font-bold text-[#ff0000] w-8 text-right">1.0x</span>
        </div>

        <!-- Toggle Orbits -->
        <button
          id="toggleOrbitsBtn"
          type="button"
          class="px-2.5 py-1.5 rounded-lg border border-[#ff0000] bg-[#ff0000]/15 text-[#ffffff] font-mono text-[11px] uppercase tracking-wider hover:bg-[#ff0000] transition-colors cursor-pointer"
        >
          ORBITS: ON
        </button>

        <!-- Toggle SFX -->
        <button
          id="toggleAudioBtn"
          type="button"
          class="px-2.5 py-1.5 rounded-lg border border-[#ffffff]/25 hover:border-[#ff0000] text-[#ffffff] font-mono text-[11px] uppercase tracking-wider transition-colors cursor-pointer"
        >
          SFX: ON
        </button>

        <!-- Solar System Overview Reset -->
        <button
          id="resetViewBtn"
          type="button"
          class="px-3 py-1.5 rounded-lg bg-[#ff0000] hover:bg-[#ff0000]/85 text-[#ffffff] font-mono font-bold text-[11px] uppercase tracking-wider shadow-[0_0_15px_rgba(255,0,0,0.5)] transition-all cursor-pointer"
        >
          SOLAR OVERVIEW
        </button>
      </div>
    </div>
  </header>

  <!-- RIGHT / BOTTOM TACTICAL INSPECTOR DRAWER (Spacecraft Photos + Structural Schematics + Explanations) -->
  <aside
    id="inspectorPanel"
    class="fixed z-30 bottom-16 sm:bottom-auto sm:top-20 right-3 left-3 sm:left-auto sm:w-[405px] lg:w-[435px] max-h-[62vh] sm:max-h-[calc(100vh-145px)] rounded-2xl border border-[#ff0000] bg-[#000000]/94 backdrop-blur-xl shadow-[0_0_40px_rgba(255,0,0,0.32)] flex flex-col overflow-hidden transition-all duration-300"
  >
    <!-- Inspector Top Header -->
    <div class="px-4 py-3 border-b border-[#ff0000]/45 bg-[#ff0000]/10 flex items-center justify-between gap-2 shrink-0">
      <div class="flex items-center gap-2 min-w-0">
        <span class="w-2.5 h-2.5 rounded-full bg-[#ff0000] animate-pulse shrink-0"></span>
        <span id="inspectorSectorBadge" class="text-[11px] font-mono font-bold uppercase tracking-widest text-[#ffffff] truncate">
          SECTOR 00 // G2V MAIN-SEQUENCE STAR
        </span>
      </div>
      <div class="flex items-center gap-1.5 shrink-0">
        <button
          id="prevBodyBtn"
          type="button"
          class="px-2 py-1 rounded border border-[#ffffff]/25 hover:border-[#ff0000] text-[11px] font-mono text-[#ffffff] cursor-pointer"
          title="Previous Celestial Object"
        >
          &larr;
        </button>
        <button
          id="nextBodyBtn"
          type="button"
          class="px-2 py-1 rounded border border-[#ffffff]/25 hover:border-[#ff0000] text-[11px] font-mono text-[#ffffff] cursor-pointer"
          title="Next Celestial Object"
        >
          &rarr;
        </button>
        <button
          id="collapseInspectorBtn"
          type="button"
          class="px-2 py-1 rounded border border-[#ff0000]/60 hover:bg-[#ff0000] text-[11px] font-mono text-[#ffffff] cursor-pointer"
          title="Minimize Inspector"
        >
          HIDE
        </button>
      </div>
    </div>

    <!-- Scrollable Content Body -->
    <div class="overflow-y-auto p-4 space-y-4">
      <!-- Visual Media Switcher Tabs (Real Spacecraft Photo vs Structural Cross-Section) -->
      <div class="flex items-center justify-between gap-2">
        <h2 id="bodyName" class="text-xl sm:text-2xl font-extrabold tracking-tight uppercase text-[#ffffff]">
          THE SUN (SOL)
        </h2>
        <div class="flex rounded-lg border border-[#ffffff]/20 p-0.5 bg-[#000000] shrink-0">
          <button
            id="viewPhotoBtn"
            type="button"
            class="px-2.5 py-1 rounded-md bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors"
          >
            REAL PHOTO
          </button>
          <button
            id="viewSchematicBtn"
            type="button"
            class="px-2.5 py-1 rounded-md text-[#ffffff]/70 hover:text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors"
          >
            STRUCTURE
          </button>
        </div>
      </div>

      <!-- Visual Viewport Frame -->
      <div class="relative rounded-xl border border-[#ff0000]/65 bg-[#000000] overflow-hidden">
        <!-- High-Tech Corner Brackets -->
        <div class="absolute top-2 left-2 w-3 h-3 border-t-2 border-l-2 border-[#ff0000] z-10 pointer-events-none"></div>
        <div class="absolute top-2 right-2 w-3 h-3 border-t-2 border-r-2 border-[#ff0000] z-10 pointer-events-none"></div>
        <div class="absolute bottom-8 left-2 w-3 h-3 border-b-2 border-l-2 border-[#ff0000] z-10 pointer-events-none"></div>
        <div class="absolute bottom-8 right-2 w-3 h-3 border-b-2 border-r-2 border-[#ff0000] z-10 pointer-events-none"></div>

        <!-- Mode A: Real Spacecraft Photography -->
        <div id="photoViewport" class="relative bg-[#000000]">
          <!-- Optical Acquisition Status Overlay -->
          <div
            id="imageLoadingOverlay"
            class="hidden absolute inset-0 z-10 bg-[#000000]/80 flex items-center justify-center pointer-events-none"
          >
            <div class="flex items-center gap-2 px-3 py-1.5 rounded border border-[#ff0000] bg-[#000000] font-mono text-[10px] text-[#ff0000] tracking-widest uppercase">
              <span class="w-2 h-2 rounded-full bg-[#ff0000] animate-ping"></span>
              <span>ACQUIRING NASA TELEMETRY...</span>
            </div>
          </div>

          <!-- NASA / ESA / JPL Mission Badge -->
          <div
            id="imageCatalogBadge"
            class="absolute top-2.5 right-6 z-10 px-2 py-0.5 rounded border border-[#ff0000]/75 bg-[#000000]/88 font-mono text-[9px] font-bold tracking-wider text-[#ffffff] uppercase pointer-events-none"
          >
            NASA // SDO ARCHIVE
          </div>

          <img
            id="bodyImage"
            src="https://commons.wikimedia.org/wiki/Special:FilePath/The_Sun_by_the_Atmospheric_Imaging_Assembly_of_NASA%27s_Solar_Dynamics_Observatory_-_20100819.jpg?width=800"
            alt="Celestial Body Spacecraft Feed"
            class="w-full h-44 sm:h-52 object-contain bg-[#000000] block contrast-105 transition-opacity duration-200"
          />
          <!-- Subtle Bottom Vignette -->
          <div class="absolute inset-0 bg-gradient-to-t from-[#000000]/75 via-transparent to-transparent pointer-events-none"></div>
        </div>

        <!-- Mode B: Custom SVG Structural Cross-Section Diagram -->
        <div id="schematicViewport" class="hidden h-44 sm:h-52 w-full bg-[#000000] flex items-center justify-center p-3">
          <!-- Custom SVG injected dynamically per body -->
        </div>

        <!-- Image Caption Footer -->
        <div class="px-3 py-1.5 bg-[#000000] border-t border-[#ffffff]/15 flex items-center justify-between text-[10px] font-mono">
          <span id="imageCaptionText" class="text-[#ff0000] font-semibold truncate">
            FIG 00 // SDO EXTREME ULTRAVIOLET CORONA
          </span>
          <span class="text-[#ffffff]/55 shrink-0">OPTICAL LOCK</span>
        </div>
      </div>

      <!-- Primary Explanation Text -->
      <div class="space-y-2">
        <div class="flex items-center justify-between gap-2">
          <div class="text-[10px] font-mono uppercase tracking-widest text-[#ff0000] font-bold">
            // ASTROPHYSICAL BRIEFING
          </div>
          <button
            id="jumpParentBtn"
            type="button"
            class="hidden px-2 py-0.5 rounded border border-[#ff0000] bg-[#ff0000]/15 hover:bg-[#ff0000] text-[#ffffff] font-mono text-[10px] uppercase cursor-pointer transition-colors"
          >
            &uarr; PARENT PLANET
          </button>
        </div>
        <p id="bodyDescription" class="text-xs sm:text-sm text-[#ffffff]/90 leading-relaxed">
          The Sun is a G2V main-sequence yellow dwarf star containing 99.86% of the Solar System's total mass. Core thermonuclear fusion converts 600 million metric tons of hydrogen into helium every second.
        </p>
      </div>

      <!-- Orbiting Moons Quick-Jump Matrix (Shown when inspecting a Planet with Moons) -->
      <div id="planetMoonsBox" class="hidden rounded-xl border border-[#ff0000]/45 bg-[#000000] p-3 space-y-2">
        <div class="text-[10px] font-mono uppercase tracking-widest text-[#ffffff]/75 flex items-center justify-between">
          <span>ORBITING SATELLITES (MOONS)</span>
          <span id="planetMoonsCount" class="text-[#ff0000] font-bold">0 MOONS</span>
        </div>
        <div id="planetMoonsList" class="flex flex-wrap gap-1.5">
          <!-- Populated dynamically -->
        </div>
      </div>

      <!-- Interactive Sub-Parts / Features Selector -->
      <div class="rounded-xl border border-[#ffffff]/15 bg-[#000000] p-3 space-y-2.5">
        <div class="text-[10px] font-mono uppercase tracking-widest text-[#ffffff]/70 flex items-center justify-between">
          <span>INSPECT SUB-SYSTEMS</span>
          <span class="text-[#ff0000]">TAP TO ANALYZE</span>
        </div>
        <div id="subFeaturesTabs" class="flex flex-wrap gap-1.5">
          <!-- Populated dynamically -->
        </div>
        <div class="pt-1 border-t border-[#ffffff]/10">
          <div id="subFeatureTitle" class="text-xs font-mono font-bold text-[#ff0000] mb-1">
            CORE FUSION REACTOR
          </div>
          <p id="subFeatureDetail" class="text-xs text-[#ffffff]/80 leading-relaxed">
            Core temperatures reach 15.7 million Kelvin under 265 billion bar of pressure, driving the proton-proton fusion chain.
          </p>
        </div>
      </div>

      <!-- Verified JPL Astronomical Telemetry Data Grid -->
      <div class="grid grid-cols-2 gap-2">
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">EQUATORIAL DIAMETER</div>
          <div id="statDiameter" class="text-xs sm:text-sm font-mono font-bold text-[#ffffff] mt-0.5">1,392,700 km</div>
        </div>
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">SEMI-MAJOR AXIS</div>
          <div id="statDistance" class="text-xs sm:text-sm font-mono font-bold text-[#ff0000] mt-0.5">0.000 AU (ORIGIN)</div>
        </div>
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">SIDEREAL PERIOD</div>
          <div id="statPeriod" class="text-xs sm:text-sm font-mono font-bold text-[#ffffff] mt-0.5">230M YRS (GALACTIC)</div>
        </div>
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">MEAN / RANGE TEMP</div>
          <div id="statTemp" class="text-xs sm:text-sm font-mono font-bold text-[#ff0000] mt-0.5">5,500 °C</div>
        </div>
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">ECCENTRICITY (e)</div>
          <div id="statEccentricity" class="text-xs sm:text-sm font-mono font-bold text-[#ffffff] mt-0.5">0.0000</div>
        </div>
        <div class="rounded-lg border border-[#ffffff]/15 p-2.5 bg-[#000000]">
          <div class="text-[10px] font-mono uppercase text-[#ffffff]/50">AXIAL OBLIQUITY</div>
          <div id="statObliquity" class="text-xs sm:text-sm font-mono font-bold text-[#ff0000] mt-0.5">7.25°</div>
        </div>
      </div>
    </div>
  </aside>

  <!-- Re-Open Inspector Button (Shown when Inspector is minimized) -->
  <button
    id="reopenInspectorBtn"
    type="button"
    class="hidden fixed z-30 bottom-16 right-4 px-4 py-2.5 rounded-xl border border-[#ff0000] bg-[#000000]/92 text-[#ffffff] font-mono text-xs font-bold uppercase tracking-wider shadow-[0_0_20px_rgba(255,0,0,0.6)] hover:bg-[#ff0000] transition-all cursor-pointer"
  >
    OPEN TELEMETRY PANEL
  </button>

  <!-- BOTTOM CELESTIAL NODE NAVIGATION BAR -->
  <nav
    aria-label="Solar System Bodies"
    class="fixed bottom-0 left-0 right-0 z-20 p-2.5 sm:p-3 pointer-events-none"
  >
    <div class="max-w-[1600px] mx-auto flex items-center justify-between gap-2">
      <!-- Category Filter + Scrollable Roster of Clickable Celestial Bodies -->
      <div class="pointer-events-auto flex items-center gap-1.5 px-2.5 py-2 rounded-xl border border-[#ffffff]/20 bg-[#000000]/90 backdrop-blur-md w-full lg:w-auto overflow-hidden">
        <!-- Quick Filter Pills -->
        <div class="flex items-center gap-1 pr-2 border-r border-[#ffffff]/15 shrink-0">
          <button
            id="filterAllBtn"
            type="button"
            class="px-2 py-1 rounded bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer"
          >
            ALL (33)
          </button>
          <button
            id="filterPlanetsBtn"
            type="button"
            class="px-2 py-1 rounded border border-[#ffffff]/20 hover:border-[#ff0000] text-[#ffffff]/80 font-mono text-[10px] uppercase cursor-pointer"
          >
            PLANETS
          </button>
          <button
            id="filterMoonsBtn"
            type="button"
            class="px-2 py-1 rounded border border-[#ffffff]/20 hover:border-[#ff0000] text-[#ffffff]/80 font-mono text-[10px] uppercase cursor-pointer"
          >
            MOONS (22)
          </button>
        </div>

        <!-- Scrollable Celestial Roster -->
        <div
          id="rosterBar"
          class="flex items-center gap-1.5 overflow-x-auto no-scrollbar"
        >
          <!-- Populated dynamically via JS -->
        </div>
      </div>

      <!-- Desktop Zoom Controls -->
      <div class="pointer-events-auto hidden lg:flex items-center gap-1.5 px-2.5 py-2 rounded-xl border border-[#ffffff]/20 bg-[#000000]/90 backdrop-blur-md shrink-0">
        <button
          id="zoomOutBtn"
          type="button"
          class="w-7 h-7 rounded border border-[#ffffff]/25 hover:border-[#ff0000] font-mono text-sm font-bold text-[#ffffff] flex items-center justify-center cursor-pointer"
          title="Zoom Out"
        >
          -
        </button>
        <span class="text-[10px] font-mono uppercase text-[#ffffff]/60 px-1">ZOOM</span>
        <button
          id="zoomInBtn"
          type="button"
          class="w-7 h-7 rounded border border-[#ffffff]/25 hover:border-[#ff0000] font-mono text-sm font-bold text-[#ffffff] flex items-center justify-center cursor-pointer"
          title="Zoom In"
        >
          +
        </button>
      </div>
    </div>
  </nav>

  <!-- Main Three.js & Interactive Solar System Engine -->
  <script>
    /**
     * SOLAR SYSTEM TACTICAL 3D SIMULATOR — COMPLETE SINGLE-FILE EDITION
     * - 33 Celestial Bodies: The Sun, 8 Planets, Ceres (Asteroid Belt), Pluto, and all 22 Major Moons
     * - Verified NASA/ESA/JPL spacecraft photography for every single planet and moon
     * - Hyper-realistic 3D spherical procedural textures, elevation bump maps, Rayleigh atmospheric scattering,
     *   1024x512 Blue Marble Earth (continents, coral shelves, clouds, city lights), and Saturn shadow-casting rings
     * - Rigid 1:1 camera lock on any chosen planet or moon
     * - Styled strictly in #000000, #ff0000, and #ffffff
     */

    // =========================================================================
    // 1. COMPLETE ASTRONOMICAL DATABASE (33 Celestial Bodies)
    // =========================================================================
    const SOLAR_BODIES = [
      // --- STAR ---
      {
        id: 'sun',
        name: 'THE SUN (SOL)',
        shortName: 'SUN',
        category: 'planet',
        sector: 'SECTOR 00 // G2V MAIN-SEQUENCE STAR',
        radius: 6.4,
        distance: 0,
        eccentricity: 0,
        inclination: 0,
        ascendingNode: 0,
        argPeriapsis: 0,
        orbitSpeed: 0,
        rotationSpeed: 0.003,
        tilt: 0.1265,
        oblateness: 0,
        bumpScale: 0,
        textureType: 'sun',
        nasaCatalog: 'NASA/SDO // AIA 171A',
        wikiFile: "The_Sun_by_the_Atmospheric_Imaging_Assembly_of_NASA's_Solar_Dynamics_Observatory_-_20100819.jpg",
        imageUrl: "https://commons.wikimedia.org/wiki/Special:FilePath/The_Sun_by_the_Atmospheric_Imaging_Assembly_of_NASA's_Solar_Dynamics_Observatory_-_20100819.jpg?width=800",
        backupImageUrl: 'https://images-assets.nasa.gov/image/GSFC_20171208_Archive_e001435/GSFC_20171208_Archive_e001435~medium.jpg',
        nasaSearchQuery: 'Solar Dynamics Observatory Sun',
        fallbackText: 'SOLAR+CORONA+OBSERVATORY+FEED',
        diameter: '1,392,700 km',
        distanceText: '0.000 AU (Barycenter)',
        periodText: '230M YRS (Galactic)',
        tempText: '5,500 °C (Photosphere)',
        eccentricityText: '0.0000',
        obliquityText: '7.25°',
        description: 'The Sun is a G2V main-sequence yellow dwarf star containing 99.86% of the Solar System’s total mass. Core thermonuclear fusion converts 600 million metric tons of hydrogen into helium every second, radiating energy across all eight planets.',
        subFeatures: [
          {
            title: 'CORE FUSION REACTOR',
            detail: 'Core temperatures reach 15.7 million Kelvin under 265 billion bar of pressure, driving the proton-proton fusion chain.'
          },
          {
            title: 'CONVECTIVE GRANULATION',
            detail: 'Boiling plasma convection cells (granules) roughly 1,000 km wide rise and sink across the photosphere every 10 minutes.'
          },
          {
            title: 'CORONAL MAGNETIC LOOPS',
            detail: 'Twisted magnetic flux tubes erupt into solar prominences and coronal mass ejections heating the outer corona above 1,000,000 K.'
          }
        ],
        schematicType: 'star'
      },

      // --- MERCURY ---
      {
        id: 'mercury',
        name: 'MERCURY',
        shortName: 'MERCURY',
        category: 'planet',
        sector: 'SECTOR 01 // INNER TERRESTRIAL PLANET',
        radius: 0.72,
        distance: 13.5,
        eccentricity: 0.2056,
        inclination: 0.122,
        ascendingNode: 0.84,
        argPeriapsis: 0.51,
        orbitSpeed: 0.016,
        rotationSpeed: 0.0025,
        tilt: 0.0005,
        oblateness: 0,
        bumpScale: 0.044,
        textureType: 'mercury',
        nasaCatalog: 'MESSENGER // MDIS MOSAIC',
        wikiFile: 'Mercury_in_true_color.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Mercury_in_true_color.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA15162/PIA15162~medium.jpg',
        nasaSearchQuery: 'Mercury MESSENGER true color',
        fallbackText: 'MERCURY+MESSENGER+ORBITAL+SCAN',
        diameter: '4,879 km',
        distanceText: '0.387 AU (57.9M km)',
        periodText: '87.97 Earth Days',
        tempText: '-180 °C to +430 °C',
        eccentricityText: '0.2056',
        obliquityText: '0.03°',
        description: 'Mercury is the smallest and innermost planet in the Solar System. Lacking a dense insulating atmosphere, it experiences the steepest temperature swings of any planet while locked in a 3:2 spin-orbit resonance.',
        subFeatures: [
          {
            title: 'CALORIS PLANITIA BASIN',
            detail: 'A 1,550 km multi-ring impact basin ringed by 2 km-high mountains, excavated 3.8 billion years ago by a 100 km asteroid.'
          },
          {
            title: 'OVERSIZED IRON CORE',
            detail: 'An unusually massive metallic core occupies ~85% of Mercury’s radius, generating a 1% offset global dipole magnetosphere.'
          },
          {
            title: 'LOBATE THRUST SCARPS',
            detail: 'Cliff-like fault scarps up to 3 km high formed as Mercury’s interior cooled and contracted globally by up to 7 km in radius.'
          }
        ],
        schematicType: 'terrestrial'
      },

      // --- VENUS ---
      {
        id: 'venus',
        name: 'VENUS',
        shortName: 'VENUS',
        category: 'planet',
        sector: 'SECTOR 02 // SUPERCRITICAL GREENHOUSE WORLD',
        radius: 1.15,
        distance: 19.5,
        eccentricity: 0.0067,
        inclination: 0.059,
        ascendingNode: 1.33,
        argPeriapsis: 0.96,
        orbitSpeed: 0.0115,
        rotationSpeed: -0.0014,
        tilt: 3.095,
        oblateness: 0,
        bumpScale: 0.012,
        atmosphere: { dayColor: [1.0, 0.84, 0.48], sunsetColor: [1.0, 0.52, 0.18], density: 0.92, power: 3.4 },
        textureType: 'venus',
        nasaCatalog: 'MARINER 10 // NASA-JPL',
        wikiFile: 'Venus-real_color.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Venus-real_color.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00271/PIA00271~medium.jpg',
        nasaSearchQuery: 'Venus Mariner 10 clouds',
        fallbackText: 'VENUS+MAGELLAN+RADAR+TOPOGRAPHY',
        diameter: '12,104 km',
        distanceText: '0.723 AU (108.2M km)',
        periodText: '224.70 Earth Days',
        tempText: '+465 °C (Mean Surface)',
        eccentricityText: '0.0067',
        obliquityText: '177.36° (Retrograde)',
        description: 'Shrouded beneath opaque sulfuric acid clouds, Venus is the hottest planet in the Solar System. Its 96.5% CO₂ atmosphere traps solar radiation under 92 bar surface pressure and rotates backward (retrograde) once every 243 Earth days.',
        subFeatures: [
          {
            title: 'SUPER-ROTATING CLOUD DECK',
            detail: 'Upper sulfuric acid clouds race around the planet at 360 km/h—circling Venus in just 4 Earth days.'
          },
          {
            title: 'MAXWELL MONTES & HIGHLANDS',
            detail: 'Volcanic mountain massifs rise 11 km above mean planetary radius across Ishtar Terra and Aphrodite Terra.'
          },
          {
            title: 'RETROGRADE AXIAL SPIN',
            detail: 'With a 177.36° axial tilt, Venus spins clockwise (east-to-west), causing the Sun to rise in the west and set in the east.'
          }
        ],
        schematicType: 'terrestrial'
      },

      // --- EARTH ---
      {
        id: 'earth',
        name: 'EARTH (TERRA)',
        shortName: 'EARTH',
        category: 'planet',
        sector: 'SECTOR 03 // BIOSPHERE OCEANIC WORLD',
        radius: 1.24,
        distance: 26.8,
        eccentricity: 0.0167,
        inclination: 0.0,
        ascendingNode: -0.19,
        argPeriapsis: 1.99,
        orbitSpeed: 0.009,
        rotationSpeed: 0.012,
        tilt: 0.4091,
        oblateness: 0.0033,
        bumpScale: 0.028,
        atmosphere: { dayColor: [0.34, 0.68, 1.0], sunsetColor: [1.0, 0.42, 0.16], density: 0.86, power: 3.8 },
        textureType: 'earth',
        hasClouds: true,
        hasCityLights: true,
        nasaCatalog: 'APOLLO 17 // BLUE MARBLE',
        wikiFile: 'The_Earth_seen_from_Apollo_17.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/The_Earth_seen_from_Apollo_17.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/GSFC_20171208_Archive_e002131/GSFC_20171208_Archive_e002131~medium.jpg',
        nasaSearchQuery: 'Earth Apollo 17 Blue Marble',
        fallbackText: 'EARTH+ORBITAL+TELEMETRY+SCAN',
        diameter: '12,742 km',
        distanceText: '1.000 AU (149.6M km)',
        periodText: '365.256 Days',
        tempText: '+15 °C (Global Mean)',
        eccentricityText: '0.0167',
        obliquityText: '23.44°',
        description: 'Earth is the largest terrestrial planet and the only known world harboring liquid surface oceans (71% coverage) and an active biosphere. Plate tectonics, a nitrogen-oxygen atmosphere, and a strong geomagnetic field sustain habitable conditions.',
        subFeatures: [
          {
            title: 'HYDROSPHERE & CORAL SHELVES',
            detail: 'Liquid water oceans moderate global climate via thermohaline circulation and absorb solar radiation across shallow continental shelves.'
          },
          {
            title: 'GEODYNAMO MAGNETOSPHERE',
            detail: 'Convecting liquid iron in the outer core generates a magnetic shield deflecting ionizing solar wind particles.'
          },
          {
            title: 'PLATE TECTONICS & ATMOSPHERE',
            detail: 'Mobile lithospheric plates recycle crustal carbon while 78% N₂ and 21% O₂ maintain surface pressure at 1.013 bar.'
          }
        ],
        schematicType: 'terrestrial'
      },

      // --- EARTH'S MOON (LUNA) ---
      {
        id: 'luna',
        name: 'THE MOON (LUNA)',
        shortName: 'LUNA',
        category: 'moon',
        parentId: 'earth',
        sector: 'SECTOR 03.1 // EARTH NATURAL SATELLITE',
        radius: 0.35,
        distance: 2.55,
        eccentricity: 0.0549,
        inclination: 0.089,
        ascendingNode: 0.4,
        argPeriapsis: 1.2,
        orbitSpeed: 0.052,
        tidallyLocked: true,
        tilt: 0.116,
        oblateness: 0,
        bumpScale: 0.036,
        textureType: 'luna',
        nasaCatalog: 'LRO // FULL MOON MOSAIC',
        wikiFile: 'FullMoon2010.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/FullMoon2010.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00405/PIA00405~medium.jpg',
        nasaSearchQuery: 'Full Moon Lunar Reconnaissance Orbiter',
        fallbackText: 'LUNA+ORBITAL+RECONNAISSANCE+SCAN',
        diameter: '3,474 km',
        distanceText: '384,400 km (From Earth)',
        periodText: '27.322 Days (Locked)',
        tempText: '-173 °C to +127 °C',
        eccentricityText: '0.0549',
        obliquityText: '6.68° (To Orbit)',
        description: 'Luna is Earth’s sole natural satellite and the fifth-largest moon in the Solar System. Tidally locked with its dark basaltic maria permanently facing Earth, it stabilizes Earth’s 23.44° axial tilt and drives ocean tides.',
        subFeatures: [
          {
            title: 'BASALTIC LUNAR MARIA',
            detail: 'Dark plains such as Mare Tranquillitatis and Oceanus Procellarum formed when ancient flood basalts filled giant impact basins.'
          },
          {
            title: 'TYCHO & COPERNICUS RAYS',
            detail: 'Young impact craters surrounded by bright radial ejecta blankets stretching thousands of kilometers across the lunar highlands.'
          },
          {
            title: 'TIDAL LOCKING & LIBRATION',
            detail: 'Gravitational tidal torque synchronized Luna’s rotation period with its 27.3-day orbital period around Earth.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- MARS ---
      {
        id: 'mars',
        name: 'MARS',
        shortName: 'MARS',
        category: 'planet',
        sector: 'SECTOR 04 // IRON-OXIDE DESERT WORLD',
        radius: 0.88,
        distance: 35.0,
        eccentricity: 0.0934,
        inclination: 0.032,
        ascendingNode: 0.86,
        argPeriapsis: 5.0,
        orbitSpeed: 0.0072,
        rotationSpeed: 0.0115,
        tilt: 0.4396,
        oblateness: 0.0058,
        bumpScale: 0.042,
        atmosphere: { dayColor: [0.95, 0.56, 0.34], sunsetColor: [0.45, 0.65, 0.95], density: 0.48, power: 4.2 },
        textureType: 'mars',
        nasaCatalog: 'ESA ROSETTA // OSIRIS TRUE COLOR',
        wikiFile: 'OSIRIS_Mars_true_color.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/OSIRIS_Mars_true_color.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA04591/PIA04591~medium.jpg',
        nasaSearchQuery: 'Mars true color Valles Marineris',
        fallbackText: 'MARS+VALLES+MARINERIS+SCAN',
        diameter: '6,779 km',
        distanceText: '1.524 AU (227.9M km)',
        periodText: '686.98 Earth Days',
        tempText: '-65 °C (Global Mean)',
        eccentricityText: '0.0934',
        obliquityText: '25.19°',
        description: 'Mars is a cold desert world colored rust-red by iron-oxide regolith. It hosts the largest shield volcanoes and deepest canyon system in the Solar System alongside seasonal water-ice and dry-ice polar caps.',
        subFeatures: [
          {
            title: 'OLYMPUS MONS SHIELD VOLCANO',
            detail: 'Standing 21.9 km high and 600 km across, Olympus Mons is nearly three times taller than Mount Everest above sea level.'
          },
          {
            title: 'VALLES MARINERIS RIFT',
            detail: 'A tectonic canyon system spanning 4,000 km along the Martian equator and plunging up to 7 km deep.'
          },
          {
            title: 'POLAR WATER & CO₂ ICE CAPS',
            detail: 'Planum Boreum and Planum Australe hold layered water ice topped by seasonal dry-ice frost that sublimates each spring.'
          }
        ],
        schematicType: 'terrestrial'
      },

      // --- PHOBOS ---
      {
        id: 'phobos',
        name: 'PHOBOS (MARS I)',
        shortName: 'PHOBOS',
        category: 'moon',
        parentId: 'mars',
        sector: 'SECTOR 04.1 // INNER MARTIAN MOON',
        radius: 0.21,
        distance: 1.55,
        eccentricity: 0.0151,
        inclination: 0.019,
        ascendingNode: 0.86,
        argPeriapsis: 2.6,
        orbitSpeed: 0.088,
        tidallyLocked: true,
        irregularSeed: 1.7,
        tilt: 0.01,
        oblateness: 0,
        bumpScale: 0.034,
        textureType: 'phobos',
        nasaCatalog: 'MRO HIRISE // PIA10368',
        wikiFile: 'Phobos_colour_2008.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Phobos_colour_2008.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA10368/PIA10368~medium.jpg',
        nasaSearchQuery: 'Phobos HiRISE Stickney',
        fallbackText: 'PHOBOS+STICKNEY+CRATER+SCAN',
        diameter: '22.2 km (Irregular)',
        distanceText: '9,376 km (From Mars)',
        periodText: '7h 39m (3x Per Sol)',
        tempText: '-40 °C (Sunlit)',
        eccentricityText: '0.0151',
        obliquityText: '0.00° (Locked)',
        description: 'Phobos is the larger and innermost of Mars’s two moons. Orbiting just 6,000 km above the Martian surface—closer than any other planetary moon—it speeds around Mars three times every Martian day and spirals inward at 1.8 meters per century.',
        subFeatures: [
          {
            title: 'STICKNEY CRATER',
            detail: 'A 9 km impact crater dominating Phobos’s leading end, fractured by linear tidal-stress grooves radiating across the moon.'
          },
          {
            title: 'ROCHE LIMIT INSPIRAL',
            detail: 'Tidal drag is slowly pulling Phobos inward; in ~50 million years it will either shatter into a Martian ring or impact Mars.'
          },
          {
            title: 'CARBONACEOUS RUBBLE PILE',
            detail: 'Low bulk density (1.87 g/cm³) indicates a porous rubble-pile interior rich in dark carbonaceous chondrite rock.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- DEIMOS ---
      {
        id: 'deimos',
        name: 'DEIMOS (MARS II)',
        shortName: 'DEIMOS',
        category: 'moon',
        parentId: 'mars',
        sector: 'SECTOR 04.2 // OUTER MARTIAN MOON',
        radius: 0.17,
        distance: 2.35,
        eccentricity: 0.0003,
        inclination: 0.031,
        ascendingNode: 0.86,
        argPeriapsis: 4.1,
        orbitSpeed: 0.042,
        tidallyLocked: true,
        irregularSeed: 4.3,
        tilt: 0.02,
        oblateness: 0,
        bumpScale: 0.02,
        textureType: 'tan-rock',
        nasaCatalog: 'MRO HIRISE // PIA11826',
        wikiFile: 'Deimos-MRO.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Deimos-MRO.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA11826/PIA11826~medium.jpg',
        nasaSearchQuery: 'Deimos Mars Reconnaissance Orbiter',
        fallbackText: 'DEIMOS+OUTER+MARTIAN+MOON',
        diameter: '12.4 km (Irregular)',
        distanceText: '23,463 km (From Mars)',
        periodText: '30.31 Hours',
        tempText: '-40 °C (Mean)',
        eccentricityText: '0.0003',
        obliquityText: '0.00° (Locked)',
        description: 'Deimos is the smaller and outermost moon of Mars. Coated in a thick, smooth blanket of fine regolith up to 100 meters deep that fills in its craters, Deimos shines like a bright star from the Martian surface.',
        subFeatures: [
          {
            title: 'REGOLITH-SMOOTHED SURFACE',
            detail: 'Fine dusty ejecta settles back onto Deimos’s low-gravity surface, softening crater rims compared to rugged Phobos.'
          },
          {
            title: 'SWIFT & VOLTAIRE CRATERS',
            detail: 'Two prominent 3 km impact craters named after literary authors who imagined Martian moons centuries before their 1877 discovery.'
          },
          {
            title: 'NEAR-CIRCULAR EQUATORIAL ORBIT',
            detail: 'Deimos maintains one of the lowest orbital eccentricities (0.00033) of any natural satellite in the Solar System.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- CERES / ASTEROID BELT ---
      {
        id: 'ceres',
        name: 'CERES & ASTEROID BELT',
        shortName: 'CERES',
        category: 'planet',
        sector: 'SECTOR 05 // MAIN ASTEROID BELT DWARF PLANET',
        radius: 0.56,
        distance: 44.5,
        eccentricity: 0.0758,
        inclination: 0.184,
        ascendingNode: 1.4,
        argPeriapsis: 1.28,
        orbitSpeed: 0.0052,
        rotationSpeed: 0.015,
        tilt: 0.07,
        oblateness: 0.06,
        bumpScale: 0.032,
        textureType: 'mercury',
        nasaCatalog: 'NASA DAWN // CERES',
        wikiFile: 'Ceres_-_RC3_-_Haulani_Crater_(22381131691)_(cropped).jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Ceres_-_RC3_-_Haulani_Crater_(22381131691)_(cropped).jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA19562/PIA19562~medium.jpg',
        nasaSearchQuery: 'Ceres Dawn Occator crater',
        fallbackText: 'CERES+AND+MAIN+ASTEROID+BELT',
        diameter: '946 km (Dwarf Planet)',
        distanceText: '2.768 AU (414.0M km)',
        periodText: '4.60 Earth Years',
        tempText: '-105 °C (Mean)',
        eccentricityText: '0.0758',
        obliquityText: '4.00°',
        description: 'Ceres is the largest object in the Main Asteroid Belt between Mars and Jupiter and the only dwarf planet inside Neptune’s orbit, accounting for roughly one-third of the entire Asteroid Belt’s mass.',
        subFeatures: [
          {
            title: 'OCCATOR BRIGHT FACULAE',
            detail: 'Brilliant white sodium carbonate salt deposits inside Occator Crater left behind by rising subsurface brine cryovolcanism.'
          },
          {
            title: 'AHUNA MONS CRYOVOLCANO',
            detail: 'A 4 km-high steep-sided dome extruded from salty muddy slurry deep within Ceres’s icy-rocky mantle.'
          },
          {
            title: 'MAIN ASTEROID BELT RESONANCE',
            detail: 'Jupiter’s immense gravity prevented planetesimals in this zone from coalescing into a fifth terrestrial planet.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- JUPITER ---
      {
        id: 'jupiter',
        name: 'JUPITER',
        shortName: 'JUPITER',
        category: 'planet',
        sector: 'SECTOR 06 // KING OF GAS GIANTS',
        radius: 3.3,
        distance: 59.0,
        eccentricity: 0.0489,
        inclination: 0.022,
        ascendingNode: 1.75,
        argPeriapsis: 4.78,
        orbitSpeed: 0.0036,
        rotationSpeed: 0.024,
        tilt: 0.0546,
        oblateness: 0.0649,
        bumpScale: 0.005,
        atmosphere: { dayColor: [0.95, 0.82, 0.65], sunsetColor: [0.85, 0.45, 0.25], density: 0.65, power: 4.0 },
        textureType: 'jupiter',
        nasaCatalog: 'HUBBLE / JUNO // TRUE COLOR',
        wikiFile: 'Jupiter_and_its_shrunken_Great_Red_Spot.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Jupiter_and_its_shrunken_Great_Red_Spot.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA22946/PIA22946~medium.jpg',
        nasaSearchQuery: 'Jupiter Great Red Spot Hubble Juno',
        fallbackText: 'JUPITER+JUNO+CYCLONE+TELEMETRY',
        diameter: '142,984 km',
        distanceText: '5.204 AU (778.5M km)',
        periodText: '11.86 Earth Years',
        tempText: '-110 °C (1-Bar Cloud Top)',
        eccentricityText: '0.0489',
        obliquityText: '3.13°',
        description: 'Jupiter is the largest planet in the Solar System—more than twice as massive as all other planets combined. Rapid 9.9-hour rotation flattens its poles by 6.5% and whips ammonia-rich clouds into counter-flowing zonal jet streams.',
        subFeatures: [
          {
            title: 'THE GREAT RED SPOT',
            detail: 'A persistent high-pressure anticyclonic storm wider than Earth, raging for centuries with 430 km/h peripheral winds.'
          },
          {
            title: 'METALLIC HYDROGEN MANTLE',
            detail: 'Under 2 million bar pressure, liquid hydrogen conducts electricity like a metal, powering a magnetosphere 20,000x stronger than Earth’s.'
          },
          {
            title: 'GALILEAN SATELLITE SYSTEM',
            detail: 'Io, Europa, Ganymede, and Callisto form a miniature planetary system locked in a 1:2:4 Laplace orbital resonance.'
          }
        ],
        schematicType: 'gas-giant'
      },

      // --- IO ---
      {
        id: 'io',
        name: 'IO (JUPITER I)',
        shortName: 'IO',
        category: 'moon',
        parentId: 'jupiter',
        sector: 'SECTOR 06.1 // GALILEAN VOLCANIC MOON',
        radius: 0.36,
        distance: 4.9,
        eccentricity: 0.0041,
        inclination: 0.055,
        ascendingNode: 1.75,
        argPeriapsis: 0.8,
        orbitSpeed: 0.068,
        tidallyLocked: true,
        tilt: 0.05,
        oblateness: 0,
        bumpScale: 0.025,
        textureType: 'io',
        nasaCatalog: 'GALILEO // PIA02308',
        wikiFile: 'Io_highest_resolution_true_color.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Io_highest_resolution_true_color.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA02308/PIA02308~medium.jpg',
        nasaSearchQuery: 'Io Galileo true color',
        fallbackText: 'IO+SULFUR+VOLCANOES+GALILEO',
        diameter: '3,643 km',
        distanceText: '421,700 km (From Jupiter)',
        periodText: '1.769 Days',
        tempText: '-130 °C (1,600 °C Lavas)',
        eccentricityText: '0.0041',
        obliquityText: '0.05° (Locked)',
        description: 'Io is the innermost Galilean moon and the most geologically active world in the Solar System. Over 400 active volcanoes coat its surface in vibrant yellow, orange, and red sulfur allotropes and silicate lava flows.',
        subFeatures: [
          {
            title: 'TIDAL FLEXING FURNACE',
            detail: 'Gravitational tugs between Jupiter, Europa, and Ganymede flex Io’s solid crust up and down by 100 meters every 42.5 hours.'
          },
          {
            title: 'PELE & LOKI PATERA PLUMES',
            detail: 'Sulfur-dioxide volcanic plumes shoot 300 to 500 km into space, feeding Jupiter’s ionized plasma torus.'
          },
          {
            title: 'ZERO IMPACT CRATERS',
            detail: 'Continuous volcanic resurfacing buries Io’s crust at ~1 cm per year, erasing every single ancient impact crater.'
          }
        ],
        schematicType: 'terrestrial'
      },

      // --- EUROPA ---
      {
        id: 'europa',
        name: 'EUROPA (JUPITER II)',
        shortName: 'EUROPA',
        category: 'moon',
        parentId: 'jupiter',
        sector: 'SECTOR 06.2 // GALILEAN SUBSURFACE OCEAN MOON',
        radius: 0.33,
        distance: 6.3,
        eccentricity: 0.009,
        inclination: 0.062,
        ascendingNode: 1.75,
        argPeriapsis: 2.1,
        orbitSpeed: 0.046,
        tidallyLocked: true,
        tilt: 0.05,
        oblateness: 0,
        bumpScale: 0.018,
        textureType: 'europa',
        nasaCatalog: 'GALILEO // PIA19048',
        wikiFile: 'Europa-moon-with-margins.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Europa-moon-with-margins.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA19048/PIA19048~medium.jpg',
        nasaSearchQuery: 'Europa Galileo ice surface',
        fallbackText: 'EUROPA+FRACTURED+ICE+SHELL',
        diameter: '3,122 km',
        distanceText: '671,034 km (From Jupiter)',
        periodText: '3.551 Days',
        tempText: '-171 °C (Surface Ice)',
        eccentricityText: '0.0090',
        obliquityText: '0.10° (Locked)',
        description: 'Europa possesses the smoothest solid surface of any known Solar System object. Beneath a 15–25 km shell of fractured water ice lies a global saltwater ocean containing more than twice the water of all Earth’s oceans.',
        subFeatures: [
          {
            title: 'GLOBAL SUBGLACIAL OCEAN',
            detail: 'A 100 km-deep liquid saltwater ocean kept warm by tidal heating in direct contact with a rocky seafloor.'
          },
          {
            title: 'CYCLOIDAL RED-BROWN LINEAE',
            detail: 'Curving double ridges stained with hydrated magnesium sulfate salts and irradiated sulfuric compounds upwelling through tidal cracks.'
          },
          {
            title: 'ASTROBIOLOGICAL TARGET',
            detail: 'Hydrothermal seafloor vents and surface oxidants make Europa a prime candidate for extraterrestrial microbial habitability.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- GANYMEDE ---
      {
        id: 'ganymede',
        name: 'GANYMEDE (JUPITER III)',
        shortName: 'GANYMEDE',
        category: 'moon',
        parentId: 'jupiter',
        sector: 'SECTOR 06.3 // LARGEST MOON IN SOLAR SYSTEM',
        radius: 0.48,
        distance: 8.1,
        eccentricity: 0.0013,
        inclination: 0.058,
        ascendingNode: 1.75,
        argPeriapsis: 3.4,
        orbitSpeed: 0.031,
        tidallyLocked: true,
        tilt: 0.05,
        oblateness: 0,
        bumpScale: 0.028,
        textureType: 'ganymede',
        nasaCatalog: 'GALILEO // PIA00716',
        wikiFile: 'Ganymede_g1_true-edit1.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Ganymede_g1_true-edit1.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00716/PIA00716~medium.jpg',
        nasaSearchQuery: 'Ganymede Galileo true color',
        fallbackText: 'GANYMEDE+GROOVED+TERRAIN+SCAN',
        diameter: '5,268 km (Larger than Mercury)',
        distanceText: '1,070,412 km (From Jupiter)',
        periodText: '7.155 Days',
        tempText: '-163 °C (Mean)',
        eccentricityText: '0.0013',
        obliquityText: '0.20° (Locked)',
        description: 'Ganymede is the largest moon in the Solar System—8% wider than the planet Mercury—and the only moon known to generate its own intrinsic dipole magnetosphere from a liquid iron core.',
        subFeatures: [
          {
            title: 'INTRINSIC MAGNETOSPHERE',
            detail: 'A convecting liquid iron-sulfide core creates an internal magnetic field nested inside Jupiter’s giant magnetosphere, producing polar auroras.'
          },
          {
            title: 'DARK REGIONES & GROOVED SULCI',
            detail: 'Ancient dark cratered terrain (Galileo Regio) is sliced by younger bright grooved ridges formed by tectonic extension.'
          },
          {
            title: 'SANDWICHED ICE-OCEAN LAYERS',
            detail: 'High-pressure ice polymorphs and liquid saltwater layers stack between Ganymede’s rocky mantle and outer ice crust.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- CALLISTO ---
      {
        id: 'callisto',
        name: 'CALLISTO (JUPITER IV)',
        shortName: 'CALLISTO',
        category: 'moon',
        parentId: 'jupiter',
        sector: 'SECTOR 06.4 // ANCIENT CRATERED GALILEAN MOON',
        radius: 0.44,
        distance: 10.4,
        eccentricity: 0.0074,
        inclination: 0.056,
        ascendingNode: 1.75,
        argPeriapsis: 5.1,
        orbitSpeed: 0.02,
        tidallyLocked: true,
        tilt: 0.05,
        oblateness: 0,
        bumpScale: 0.036,
        textureType: 'callisto',
        nasaCatalog: 'GALILEO // PIA03456',
        wikiFile: 'Callisto.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Callisto.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA03456/PIA03456~medium.jpg',
        nasaSearchQuery: 'Callisto Galileo mosaic',
        fallbackText: 'CALLISTO+VALHALLA+IMPACT+BASIN',
        diameter: '4,821 km',
        distanceText: '1,882,709 km (From Jupiter)',
        periodText: '16.689 Days',
        tempText: '-139 °C (Mean)',
        eccentricityText: '0.0074',
        obliquityText: '0.19° (Locked)',
        description: 'Callisto is the outermost Galilean moon and preserves the oldest, most heavily cratered surface in the Solar System. Orbiting outside Jupiter’s intense radiation belts, it is considered an ideal staging base for future outer-system exploration.',
        subFeatures: [
          {
            title: 'VALHALLA MULTI-RING BASIN',
            detail: 'A colossal 3,800 km concentric ring structure formed when a giant impactor struck Callisto’s ancient icy-rocky crust.'
          },
          {
            title: 'UNDIFFERENTIATED ROCK-ICE INTERIOR',
            detail: 'Unlike Io, Europa, and Ganymede, Callisto never experienced strong tidal heating, leaving its rock and ice only partially separated.'
          },
          {
            title: 'LOW-RADIATION ORBITAL HAVEN',
            detail: 'At 1.88 million km from Jupiter, surface radiation is over 1,000 times lower than on Europa.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- SATURN ---
      {
        id: 'saturn',
        name: 'SATURN',
        shortName: 'SATURN',
        category: 'planet',
        sector: 'SECTOR 07 // RINGED GAS GIANT SYSTEM',
        radius: 2.75,
        distance: 84.0,
        eccentricity: 0.0565,
        inclination: 0.043,
        ascendingNode: 1.98,
        argPeriapsis: 5.91,
        orbitSpeed: 0.0024,
        rotationSpeed: 0.022,
        tilt: 0.4665,
        oblateness: 0.0979,
        bumpScale: 0.004,
        hasRings: true,
        ringInner: 3.45,
        ringOuter: 6.45,
        atmosphere: { dayColor: [0.96, 0.86, 0.62], sunsetColor: [0.88, 0.58, 0.32], density: 0.62, power: 4.0 },
        textureType: 'saturn',
        nasaCatalog: 'CASSINI // EQUINOX MOSAIC',
        wikiFile: 'Saturn_during_Equinox.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Saturn_during_Equinox.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA21046/PIA21046~medium.jpg',
        nasaSearchQuery: 'Saturn Cassini Equinox rings',
        fallbackText: 'SATURN+CASSINI+RING+SYSTEM+SCAN',
        diameter: '120,536 km (282,000 km Rings)',
        distanceText: '9.582 AU (1.43B km)',
        periodText: '29.45 Earth Years',
        tempText: '-140 °C (1-Bar Cloud Top)',
        eccentricityText: '0.0565',
        obliquityText: '26.73°',
        description: 'Saturn is the second-largest planet and the jewel of the Solar System, encircled by an immense ring system spanning 282,000 km yet averaging just 10 meters thick. With a mean density of 0.687 g/cm³, Saturn is less dense than liquid water.',
        subFeatures: [
          {
            title: 'D, C, B, CASSINI & A RING ARCHITECTURE',
            detail: 'Billions of 99.9% pure water-ice particles sculpted by shepherd moons and the 4,800 km Cassini Division cleared by Mimas.'
          },
          {
            title: 'NORTH POLAR HEXAGON JET STREAM',
            detail: 'A persistent six-sided Rossby wave jet stream 29,000 km wide rotating around Saturn’s north pole.'
          },
          {
            title: 'EXTREME 9.8% POLAR OBLATENESS',
            detail: 'Low bulk density combined with a 10.7-hour spin visibly bulges Saturn’s equator by nearly 12,000 km compared to its poles.'
          }
        ],
        schematicType: 'ringed'
      },

      // --- MIMAS ---
      {
        id: 'mimas',
        name: 'MIMAS (SATURN I)',
        shortName: 'MIMAS',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.1 // INNERMOST MAJOR SATURNIAN MOON',
        radius: 0.2,
        distance: 7.05,
        eccentricity: 0.0196,
        inclination: 0.493,
        ascendingNode: 1.98,
        argPeriapsis: 0.5,
        orbitSpeed: 0.072,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.04,
        textureType: 'mimas',
        nasaCatalog: 'CASSINI // PIA12570',
        wikiFile: 'Mimas_Cassini.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Mimas_Cassini.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA12570/PIA12570~medium.jpg',
        nasaSearchQuery: 'Mimas Cassini Herschel crater',
        fallbackText: 'MIMAS+HERSCHEL+CRATER+CASSINI',
        diameter: '396 km',
        distanceText: '185,539 km (From Saturn)',
        periodText: '22.6 Hours',
        tempText: '-209 °C (Mean)',
        eccentricityText: '0.0196',
        obliquityText: '0.00° (Locked)',
        description: 'Mimas is the smallest astronomical body known to have pulled itself into a rounded hydrostatic shape by self-gravity. Dominated by the colossal 139 km Herschel Crater, it clears the Cassini Division in Saturn’s rings via a 2:1 orbital resonance.',
        subFeatures: [
          {
            title: 'HERSCHEL IMPACT CRATER',
            detail: 'Measuring one-third of Mimas’s entire diameter with 5 km walls and a 6 km central peak, the impact nearly shattered the moon.'
          },
          {
            title: 'CASSINI DIVISION SHEPHERDING',
            detail: 'Mimas’s 2:1 resonance sweeps ice particles out of the 4,800 km gap separating Saturn’s A and B rings.'
          },
          {
            title: 'YOUNG SUBSURFACE OCEAN CANDIDATE',
            detail: 'Cassini orbital libration measurements suggest a geologically young liquid ocean hidden 20–30 km beneath Mimas’s cratered ice.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- ENCELADUS ---
      {
        id: 'enceladus',
        name: 'ENCELADUS (SATURN II)',
        shortName: 'ENCELADUS',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.2 // CRYOVOLCANIC GEYSER MOON',
        radius: 0.23,
        distance: 8.1,
        eccentricity: 0.0047,
        inclination: 0.467,
        ascendingNode: 1.98,
        argPeriapsis: 1.4,
        orbitSpeed: 0.056,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.022,
        textureType: 'enceladus',
        nasaCatalog: 'CASSINI // PIA17202',
        wikiFile: 'Enceladus_stripes_crop.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Enceladus_stripes_crop.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA17202/PIA17202~medium.jpg',
        nasaSearchQuery: 'Enceladus Cassini tiger stripes',
        fallbackText: 'ENCELADUS+TIGER+STRIPES+PLUMES',
        diameter: '504 km',
        distanceText: '238,037 km (From Saturn)',
        periodText: '1.370 Days',
        tempText: '-198 °C (-75 °C Fractures)',
        eccentricityText: '0.0047',
        obliquityText: '0.00° (Locked)',
        description: 'Enceladus is the most reflective world in the Solar System, coated in fresh snow-white water ice. Over 100 cryovolcanic geysers erupt from warm south-polar fractures, jetting ocean water, salts, silica, and organic molecules into space to form Saturn’s E Ring.',
        subFeatures: [
          {
            title: 'TIGER STRIPE CRYOVOLCANIC VENTS',
            detail: 'Four parallel tectonic fissures (Alexandria, Cairo, Baghdad, and Damascus Sulci) blast supersonic water-ice plumes at 2,189 km/h.'
          },
          {
            title: 'HYDROTHERMAL SEAFLOOR OCEAN',
            detail: 'Cassini detected molecular hydrogen (H₂) and nanosilica grains in the plumes—direct evidence of active hydrothermal vents on the ocean floor.'
          },
          {
            title: 'SATURN E-RING FEEDER',
            detail: 'Escaping ice crystals replenish Saturn’s vast outer diffuse E Ring and coat neighboring moons in bright fresh frost.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- TETHYS ---
      {
        id: 'tethys',
        name: 'TETHYS (SATURN III)',
        shortName: 'TETHYS',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.3 // MID-SIZED ICY SATURNIAN MOON',
        radius: 0.27,
        distance: 9.25,
        eccentricity: 0.0001,
        inclination: 0.486,
        ascendingNode: 1.98,
        argPeriapsis: 2.3,
        orbitSpeed: 0.044,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.03,
        textureType: 'tethys',
        nasaCatalog: 'CASSINI // PIA18317',
        wikiFile: 'PIA18317-SaturnMoon-Tethys-Cassini-20150411.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/PIA18317-SaturnMoon-Tethys-Cassini-20150411.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA18317/PIA18317~medium.jpg',
        nasaSearchQuery: 'Tethys Cassini',
        fallbackText: 'TETHYS+ITHACA+CHASMA+SCAN',
        diameter: '1,062 km',
        distanceText: '294,619 km (From Saturn)',
        periodText: '1.888 Days',
        tempText: '-187 °C (Mean)',
        eccentricityText: '0.0001',
        obliquityText: '0.00° (Locked)',
        description: 'Tethys is composed almost entirely of pure crystalline water ice with a density of just 0.98 g/cm³. It is scarred by the massive 450 km Odysseus Crater and a planet-spanning canyon system called Ithaca Chasma.',
        subFeatures: [
          {
            title: 'ITHACA CHASMA GRABEN',
            detail: 'A giant rift valley 100 km wide, 3–5 km deep, and 2,000 km long—stretching across three-quarters of Tethys’s circumference.'
          },
          {
            title: 'ODYSSEUS IMPACT BASIN',
            detail: 'A 450 km shallow relaxed crater on Tethys’s leading hemisphere whose floor has rebounded to match the moon’s curvature.'
          },
          {
            title: 'TROJAN CO-ORBITAL MOONS',
            detail: 'Two small moons, Telesto and Calypso, orbit in Tethys’s stable L4 and L5 Lagrange points 60° ahead and behind.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- DIONE ---
      {
        id: 'dione',
        name: 'DIONE (SATURN IV)',
        shortName: 'DIONE',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.4 // FRACTURED ICE-CLIFF MOON',
        radius: 0.28,
        distance: 10.5,
        eccentricity: 0.0022,
        inclination: 0.467,
        ascendingNode: 1.98,
        argPeriapsis: 3.3,
        orbitSpeed: 0.035,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.026,
        textureType: 'icy-grey',
        nasaCatalog: 'CASSINI // NATURAL LIGHT',
        wikiFile: 'Dione_in_natural_light_(cropped).jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Dione_in_natural_light_(cropped).jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA07746/PIA07746~medium.jpg',
        nasaSearchQuery: 'Dione Cassini natural light',
        fallbackText: 'DIONE+ICE+CLIFFS+CASSINI',
        diameter: '1,123 km',
        distanceText: '377,396 km (From Saturn)',
        periodText: '2.737 Days',
        tempText: '-186 °C (Mean)',
        eccentricityText: '0.0022',
        obliquityText: '0.00° (Locked)',
        description: 'Dione is a dense ice-and-silicate moon featuring bright tectonic ice cliffs (chasmata) slicing across its dark trailing hemisphere. Gravity data from Cassini indicates a subsurface liquid ocean ~100 km below its crust.',
        subFeatures: [
          {
            title: 'BRIGHT TECTONIC ICE CLIFFS',
            detail: 'Braided networks of 100-to-1,000 meter tall vertical water-ice fault scarps expose fresh reflective ice along Dione’s trailing face.'
          },
          {
            title: 'SILICATE-RICH CORE',
            detail: 'At 1.48 g/cm³, Dione is denser than Tethys and Rhea, indicating a substantial rocky silicate core inside its icy mantle.'
          },
          {
            title: 'ENCELADUS 1:2 ORBITAL RESONANCE',
            detail: 'Dione’s gravitational resonance pumps orbital eccentricity into Enceladus, fueling Enceladus’s active tidal geysers.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- RHEA ---
      {
        id: 'rhea',
        name: 'RHEA (SATURN V)',
        shortName: 'RHEA',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.5 // SECOND-LARGEST SATURNIAN MOON',
        radius: 0.34,
        distance: 12.1,
        eccentricity: 0.0012,
        inclination: 0.472,
        ascendingNode: 1.98,
        argPeriapsis: 4.4,
        orbitSpeed: 0.026,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.032,
        textureType: 'icy-grey',
        nasaCatalog: 'CASSINI // PIA07763',
        wikiFile: 'PIA07763_Rhea_full_globe5.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/PIA07763_Rhea_full_globe5.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA07763/PIA07763~medium.jpg',
        nasaSearchQuery: 'Rhea Cassini full globe',
        fallbackText: 'RHEA+CRATERED+ICE+SPHERE',
        diameter: '1,527 km',
        distanceText: '527,108 km (From Saturn)',
        periodText: '4.518 Days',
        tempText: '-174 °C to -220 °C',
        eccentricityText: '0.0012',
        obliquityText: '0.00° (Locked)',
        description: 'Rhea is Saturn’s second-largest moon, composed of roughly 75% water ice and 25% rock. Its heavily cratered highlands are punctuated by bright white ray craters and a tenuous oxygen–carbon dioxide exosphere.',
        subFeatures: [
          {
            title: 'INKTOMI RAY CRATER',
            detail: 'A geologically young 48 km crater surrounded by a dazzling butterfly-shaped blanket of bright excavated water ice.'
          },
          {
            title: 'TENUOUS O₂ / CO₂ EXOSPHERE',
            detail: 'Saturn’s magnetospheric ions sputter water molecules from Rhea’s surface ice, sustaining a thin oxygen and carbon-dioxide atmosphere.'
          },
          {
            title: 'HIGHLAND CRATER SATURATION',
            detail: 'Ancient terrain saturated with overlapping multi-ring craters up to 480 km across (Tirawa Basin).'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- TITAN ---
      {
        id: 'titan',
        name: 'TITAN (SATURN VI)',
        shortName: 'TITAN',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.6 // HYDROCARBON OCEAN & HAZE WORLD',
        radius: 0.5,
        distance: 14.5,
        eccentricity: 0.0288,
        inclination: 0.472,
        ascendingNode: 1.98,
        argPeriapsis: 5.5,
        orbitSpeed: 0.017,
        tidallyLocked: true,
        tilt: 0.4665,
        oblateness: 0,
        bumpScale: 0.012,
        atmosphere: { dayColor: [0.96, 0.62, 0.22], sunsetColor: [0.38, 0.62, 0.98], density: 1.15, power: 3.1 },
        textureType: 'titan',
        nasaCatalog: 'CASSINI // TRUE COLOR',
        wikiFile: 'Titan_in_true_color.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Titan_in_true_color.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA20016/PIA20016~medium.jpg',
        nasaSearchQuery: 'Titan Cassini true color atmosphere',
        fallbackText: 'TITAN+ORANGE+METHANE+HAZE',
        diameter: '5,149 km (Larger than Mercury)',
        distanceText: '1,221,870 km (From Saturn)',
        periodText: '15.945 Days',
        tempText: '-179.5 °C (Surface)',
        eccentricityText: '0.0288',
        obliquityText: '0.00° (Locked)',
        description: 'Titan is Saturn’s largest moon and the only moon in the Solar System with a dense, hazy atmosphere (1.45 bar surface pressure—denser than Earth’s). Beneath golden organic smog lie rivers, rain, and seas of liquid methane and ethane.',
        subFeatures: [
          {
            title: 'KRAKEN & LIGEIA MARE HYDROCARBON SEAS',
            detail: 'Vast polar seas of liquid methane and ethane up to 300 meters deep fed by dendritic river channels and methane rainstorms.'
          },
          {
            title: 'DENSE NITROGEN-METHANE ATMOSPHERE',
            detail: '95% nitrogen and 5% methane irradiated by solar UV light creates a thick golden tholin haze with a blue-violet upper detached layer.'
          },
          {
            title: 'HUYGENS LANDING & WATER-ICE BEDROCK',
            detail: 'At -179.5 °C, water ice freezes as hard as granite, forming river-rounded cobbles photographed on the surface by ESA’s Huygens probe.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- IAPETUS ---
      {
        id: 'iapetus',
        name: 'IAPETUS (SATURN VIII)',
        shortName: 'IAPETUS',
        category: 'moon',
        parentId: 'saturn',
        sector: 'SECTOR 07.7 // TWO-TONE EQUATORIAL RIDGE MOON',
        radius: 0.33,
        distance: 17.6,
        eccentricity: 0.0286,
        inclination: 0.27,
        ascendingNode: 2.4,
        argPeriapsis: 1.9,
        orbitSpeed: 0.011,
        tidallyLocked: true,
        tilt: 0.27,
        oblateness: 0.04,
        bumpScale: 0.036,
        textureType: 'iapetus',
        nasaCatalog: 'CASSINI // PIA08384',
        wikiFile: 'Iapetus_706_1419_1.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Iapetus_706_1419_1.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA08384/PIA08384~medium.jpg',
        nasaSearchQuery: 'Iapetus Cassini two tone ridge',
        fallbackText: 'IAPETUS+TWO+TONE+YIN+YANG+MOON',
        diameter: '1,469 km',
        distanceText: '3,560,820 km (From Saturn)',
        periodText: '79.321 Days',
        tempText: '-143 °C (Dark) / -180 °C (Bright)',
        eccentricityText: '0.0286',
        obliquityText: '0.00° (Locked)',
        description: 'Iapetus is famous for its dramatic "yin-yang" two-tone coloration—a pitch-black leading hemisphere (Cassini Regio) contrasting against a dazzling white icy trailing hemisphere—and a towering 20 km-high mountain ridge running along its equator.',
        subFeatures: [
          {
            title: '20 KM EQUATORIAL MOUNTAIN RIDGE',
            detail: 'A colossal 1,300 km chain of peaks rising up to 20 km high directly along the equator, giving Iapetus the profile of a walnut.'
          },
          {
            title: 'CASSINI REGIO THERMAL SEGREGATION',
            detail: 'Dark dust swept from outer moon Phoebe absorbs sunlight, sublimating water ice that migrates and re-freezes onto the bright poles and trailing face.'
          },
          {
            title: 'HIGHLY INCLINED OUTER ORBIT',
            detail: 'Unlike Saturn’s inner moons, Iapetus orbits at a 15.5° inclination, offering sweeping panoramic views of Saturn’s ring plane.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- URANUS ---
      {
        id: 'uranus',
        name: 'URANUS',
        shortName: 'URANUS',
        category: 'planet',
        sector: 'SECTOR 08 // AXIALLY TILTED ICE GIANT',
        radius: 1.85,
        distance: 112.0,
        eccentricity: 0.0457,
        inclination: 0.013,
        ascendingNode: 1.29,
        argPeriapsis: 1.68,
        orbitSpeed: 0.0017,
        rotationSpeed: -0.018,
        tilt: 1.7064,
        oblateness: 0.0229,
        bumpScale: 0.002,
        hasRings: true,
        ringInner: 2.35,
        ringOuter: 3.05,
        ringTint: 'uranus',
        atmosphere: { dayColor: [0.55, 0.92, 0.98], sunsetColor: [0.35, 0.72, 0.88], density: 0.72, power: 3.8 },
        textureType: 'uranus',
        nasaCatalog: 'VOYAGER 2 // PIA18182',
        wikiFile: 'Uranus2.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Uranus2.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA18182/PIA18182~medium.jpg',
        nasaSearchQuery: 'Uranus Voyager 2',
        fallbackText: 'URANUS+VOYAGER+2+ICE+GIANT+SCAN',
        diameter: '51,118 km',
        distanceText: '19.191 AU (2.87B km)',
        periodText: '84.02 Earth Years',
        tempText: '-224 °C (Coldest Planet)',
        eccentricityText: '0.0457',
        obliquityText: '97.77° (Sideways)',
        description: 'Uranus is an ice giant whose rotation axis is tipped 97.77° onto its side—rolling around the Sun like a tilted gyroscope with 42-year-long polar summers and winters. Atmospheric methane gas absorbs red light, giving Uranus its pale cyan hue.',
        subFeatures: [
          {
            title: '97.77° EXTREME AXIAL OBLIQUITY',
            detail: 'Likely caused by a colossal protoplanetary collision, tilting Uranus’s 13 narrow dark rings and 28 moons sideways.'
          },
          {
            title: 'SUPERIONIC WATER-AMMONIA MANTLE',
            detail: 'Beneath hydrogen-helium clouds lies a hot, dense fluid mantle of water, ammonia, and methane ices.'
          },
          {
            title: 'OFFSET TUMBLED MAGNETOSPHERE',
            detail: 'Uranus’s magnetic field is tilted 59° away from its rotational axis and offset from the planet’s center.'
          }
        ],
        schematicType: 'ice-giant'
      },

      // --- MIRANDA ---
      {
        id: 'miranda',
        name: 'MIRANDA (URANUS V)',
        shortName: 'MIRANDA',
        category: 'moon',
        parentId: 'uranus',
        sector: 'SECTOR 08.1 // EXTREME CLIFF URANIAN MOON',
        radius: 0.21,
        distance: 3.45,
        eccentricity: 0.0013,
        inclination: 1.78,
        ascendingNode: 1.29,
        argPeriapsis: 0.7,
        orbitSpeed: 0.064,
        tidallyLocked: true,
        tilt: 1.7064,
        oblateness: 0,
        bumpScale: 0.032,
        textureType: 'europa',
        nasaCatalog: 'VOYAGER 2 // PIA18185',
        wikiFile: 'Miranda_mosaic_in_color_-_Voyager_2.png',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Miranda_mosaic_in_color_-_Voyager_2.png?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA18185/PIA18185~medium.jpg',
        nasaSearchQuery: 'Miranda Voyager 2 mosaic',
        fallbackText: 'MIRANDA+VERONA+RUPES+CLIFF',
        diameter: '472 km',
        distanceText: '129,390 km (From Uranus)',
        periodText: '1.413 Days',
        tempText: '-213 °C (Mean)',
        eccentricityText: '0.0013',
        obliquityText: '0.00° (Locked)',
        description: 'Miranda is the smallest and innermost of Uranus’s five major moons, boasting the most extreme patchwork geology in the Solar System—including Verona Rupes, a sheer tectonic ice cliff 20 kilometers high.',
        subFeatures: [
          {
            title: 'VERONA RUPES (20 KM SHEER CLIFF)',
            detail: 'The tallest known vertical cliff in the Solar System; in Miranda’s low 0.079 m/s² gravity, a falling object would take 12 minutes to reach the bottom.'
          },
          {
            title: 'CHEVRON & CORONAE TECTONIC BLOCKS',
            detail: 'Giant racetrack-like concentric ridges (Arden, Inverness, and Elsinore Coronae) driven by past tidal-resonance diapir upwelling.'
          },
          {
            title: 'SIDEWAYS URANIAN RING PLANE ORBIT',
            detail: 'Orbits directly in Uranus’s 97.77° tilted equatorial plane alongside the planet’s dark ring system.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- ARIEL ---
      {
        id: 'ariel',
        name: 'ARIEL (URANUS I)',
        shortName: 'ARIEL',
        category: 'moon',
        parentId: 'uranus',
        sector: 'SECTOR 08.2 // BRIGHTEST URANIAN MOON',
        radius: 0.28,
        distance: 4.35,
        eccentricity: 0.0012,
        inclination: 1.71,
        ascendingNode: 1.29,
        argPeriapsis: 1.9,
        orbitSpeed: 0.048,
        tidallyLocked: true,
        tilt: 1.7064,
        oblateness: 0,
        bumpScale: 0.025,
        textureType: 'icy-grey',
        nasaCatalog: 'VOYAGER 2 // PIA01534',
        wikiFile: 'Ariel_(moon).jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Ariel_(moon).jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA01534/PIA01534~medium.jpg',
        nasaSearchQuery: 'Ariel moon Voyager 2',
        fallbackText: 'ARIEL+GRABEN+VALLEYS+VOYAGER',
        diameter: '1,158 km',
        distanceText: '190,900 km (From Uranus)',
        periodText: '2.520 Days',
        tempText: '-213 °C (Mean)',
        eccentricityText: '0.0012',
        obliquityText: '0.00° (Locked)',
        description: 'Ariel has the brightest and youngest surface among Uranus’s moons, crisscrossed by sweeping fault canyons and smooth cryovolcanic valley floors that erased most ancient craters.',
        subFeatures: [
          {
            title: 'KACHINA CHASMATA RIFT SYSTEM',
            detail: 'A 620 km-long system of parallel fault grabens whose smooth floors were flooded by viscous ammonia-water cryolavas.'
          },
          {
            title: 'FRESH WATER-ICE & CO₂ FROST',
            detail: 'Spectroscopy reveals crystalline water ice and seasonal dry-ice frost deposits across Ariel’s trailing hemisphere.'
          },
          {
            title: 'PAST TIDAL RESONANCE HEATING',
            detail: 'Ancient orbital resonances with Miranda and Umbriel melted Ariel’s interior and drove widespread crustal extension.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- UMBRIEL ---
      {
        id: 'umbriel',
        name: 'UMBRIEL (URANUS II)',
        shortName: 'UMBRIEL',
        category: 'moon',
        parentId: 'uranus',
        sector: 'SECTOR 08.3 // DARKEST MAJOR URANIAN MOON',
        radius: 0.28,
        distance: 5.35,
        eccentricity: 0.0039,
        inclination: 1.71,
        ascendingNode: 1.29,
        argPeriapsis: 3.1,
        orbitSpeed: 0.036,
        tidallyLocked: true,
        tilt: 1.7064,
        oblateness: 0,
        bumpScale: 0.03,
        textureType: 'umbriel',
        nasaCatalog: 'VOYAGER 2 // PIA00040',
        wikiFile: 'PIA00040_Umbrielx2.47.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/PIA00040_Umbrielx2.47.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00040/PIA00040~medium.jpg',
        nasaSearchQuery: 'Umbriel Voyager 2 Wunda',
        fallbackText: 'UMBRIEL+WUNDA+BRIGHT+RING',
        diameter: '1,169 km',
        distanceText: '266,000 km (From Uranus)',
        periodText: '4.144 Days',
        tempText: '-198 °C (Mean)',
        eccentricityText: '0.0039',
        obliquityText: '0.00° (Locked)',
        description: 'Umbriel is the darkest of Uranus’s major moons, reflecting less than half as much light as Ariel. Its ancient, heavily cratered surface stands out for a mysterious fluorescent white ring inside Wunda Crater near its equator.',
        subFeatures: [
          {
            title: 'WUNDA CRATER BRIGHT RING',
            detail: 'A 131 km crater near Umbriel’s equator floored by a bright 10 km-wide ring of pure ice or carbon dioxide frost.'
          },
          {
            title: 'DARK RADIATION-PROCESSED MANTLE',
            detail: 'Cosmic rays and magnetospheric particles darkened methane-bearing surface ices over 4 billion years without resurfacing.'
          },
          {
            title: '50-50 ROCK AND ICE COMPOSITION',
            detail: 'Composed of roughly equal parts water ice and dense silicate rock surrounding a small rocky core.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- TITANIA ---
      {
        id: 'titania',
        name: 'TITANIA (URANUS III)',
        shortName: 'TITANIA',
        category: 'moon',
        parentId: 'uranus',
        sector: 'SECTOR 08.4 // LARGEST URANIAN MOON',
        radius: 0.34,
        distance: 6.5,
        eccentricity: 0.0011,
        inclination: 1.71,
        ascendingNode: 1.29,
        argPeriapsis: 4.3,
        orbitSpeed: 0.025,
        tidallyLocked: true,
        tilt: 1.7064,
        oblateness: 0,
        bumpScale: 0.03,
        textureType: 'tethys',
        nasaCatalog: 'VOYAGER 2 // PIA00039',
        wikiFile: 'Titania_(moon)_color,_cropped.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Titania_(moon)_color,_cropped.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00039/PIA00039~medium.jpg',
        nasaSearchQuery: 'Titania moon Voyager 2',
        fallbackText: 'TITANIA+MESSINA+CHASMA+SCAN',
        diameter: '1,578 km',
        distanceText: '435,910 km (From Uranus)',
        periodText: '8.706 Days',
        tempText: '-203 °C (Mean)',
        eccentricityText: '0.0011',
        obliquityText: '0.00° (Locked)',
        description: 'Titania is the largest moon of Uranus and the eighth-largest moon in the Solar System. Its brownish-grey icy crust is split by Messina Chasma, a 1,500 km tectonic canyon system formed as Titania’s interior froze and expanded.',
        subFeatures: [
          {
            title: 'MESSINA CHASMA FAULT SYSTEM',
            detail: 'Twin normal-fault scarps stretching 1,500 km from the equator nearly to the south pole, slicing cleanly across older impact craters.'
          },
          {
            title: 'GERTRUDE IMPACT BASIN',
            detail: 'A 326 km degraded multi-ring impact basin surrounded by brighter icy ejecta blankets.'
          },
          {
            title: 'POSSIBLE RESIDUAL LIQUID LAYER',
            detail: 'Ammonia antifreeze dissolved in water could sustain a thin liquid ocean layer between Titania’s rocky core and icy mantle.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- OBERON ---
      {
        id: 'oberon',
        name: 'OBERON (URANUS IV)',
        shortName: 'OBERON',
        category: 'moon',
        parentId: 'uranus',
        sector: 'SECTOR 08.5 // OUTERMOST MAJOR URANIAN MOON',
        radius: 0.33,
        distance: 7.8,
        eccentricity: 0.0014,
        inclination: 1.71,
        ascendingNode: 1.29,
        argPeriapsis: 5.5,
        orbitSpeed: 0.017,
        tidallyLocked: true,
        tilt: 1.7064,
        oblateness: 0,
        bumpScale: 0.032,
        textureType: 'callisto',
        nasaCatalog: 'VOYAGER 2 // PIA00034',
        wikiFile: 'Voyager_2_picture_of_Oberon.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Voyager_2_picture_of_Oberon.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00034/PIA00034~medium.jpg',
        nasaSearchQuery: 'Oberon Voyager 2',
        fallbackText: 'OBERON+HAMLET+CRATER+VOYAGER',
        diameter: '1,523 km',
        distanceText: '583,520 km (From Uranus)',
        periodText: '13.463 Days',
        tempText: '-198 °C (Mean)',
        eccentricityText: '0.0014',
        obliquityText: '0.00° (Locked)',
        description: 'Oberon is the outermost and second-largest major moon of Uranus. Its ancient reddish-tinted icy surface is saturated with large craters whose floors are covered in mysterious dark carbonaceous material.',
        subFeatures: [
          {
            title: 'HAMLET & OTHELLO DARK-FLOORED CRATERS',
            detail: 'Large 200+ km craters surrounded by bright white ice rays yet floored with dark carbon-rich deposits upwelling from below.'
          },
          {
            title: '11 KM LIMB MOUNTAIN PEAK',
            detail: 'Voyager 2 spotted an 11 km-high mountain jutting above Oberon’s limb—likely the central peak of an immense ancient basin.'
          },
          {
            title: 'OUTER MAGNETOSPHERIC BOUNDARY',
            detail: 'Oberon spends part of its 13.5-day orbit outside Uranus’s magnetosphere, directly exposed to the solar wind.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- NEPTUNE ---
      {
        id: 'neptune',
        name: 'NEPTUNE',
        shortName: 'NEPTUNE',
        category: 'planet',
        sector: 'SECTOR 09 // SUPERSONIC AZURE ICE GIANT',
        radius: 1.8,
        distance: 142.0,
        eccentricity: 0.0113,
        inclination: 0.031,
        ascendingNode: 2.29,
        argPeriapsis: 4.76,
        orbitSpeed: 0.0012,
        rotationSpeed: 0.019,
        tilt: 0.4943,
        oblateness: 0.0171,
        bumpScale: 0.003,
        atmosphere: { dayColor: [0.22, 0.52, 1.0], sunsetColor: [0.42, 0.72, 1.0], density: 0.85, power: 3.6 },
        textureType: 'neptune',
        nasaCatalog: 'VOYAGER 2 // PIA01492',
        wikiFile: 'Neptune_Full.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Neptune_Full.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA01492/PIA01492~medium.jpg',
        nasaSearchQuery: 'Neptune Voyager 2 full disk',
        fallbackText: 'NEPTUNE+VOYAGER+2+AZURE+ATMOSPHERE',
        diameter: '49,528 km',
        distanceText: '30.07 AU (4.50B km)',
        periodText: '164.8 Earth Years',
        tempText: '-214 °C (1-Bar Cloud Top)',
        eccentricityText: '0.0113',
        obliquityText: '28.32°',
        description: 'Neptune is the outermost major planet in the Solar System, orbiting 30 times farther from the Sun than Earth. Driven by strong internal heat, Neptune generates the fastest sustained winds in the Solar System—exceeding 2,100 km/h.',
        subFeatures: [
          {
            title: '2,100 KM/H SUPERSONIC JET STREAMS',
            detail: 'Dark anticyclonic vortices and high-altitude white methane-ice cirrus streaks whip westward across Neptune’s azure troposphere.'
          },
          {
            title: 'THE GREAT DARK SPOT & SCOOTER',
            detail: 'Earth-sized dark storm ovals form and dissipate over several years, accompanied by bright high-altitude companion clouds.'
          },
          {
            title: 'INTERNAL HEAT ENGINE',
            detail: 'Neptune radiates 2.61 times as much energy as it receives from the distant Sun, powering vigorous atmospheric convection.'
          }
        ],
        schematicType: 'ice-giant'
      },

      // --- PROTEUS ---
      {
        id: 'proteus',
        name: 'PROTEUS (NEPTUNE VIII)',
        shortName: 'PROTEUS',
        category: 'moon',
        parentId: 'neptune',
        sector: 'SECTOR 09.1 // INNER NEPTUNIAN MOON',
        radius: 0.2,
        distance: 2.75,
        eccentricity: 0.0005,
        inclination: 0.495,
        ascendingNode: 2.29,
        argPeriapsis: 1.2,
        orbitSpeed: 0.068,
        tidallyLocked: true,
        irregularSeed: 7.2,
        tilt: 0.4943,
        oblateness: 0,
        bumpScale: 0.03,
        textureType: 'dark-rock',
        nasaCatalog: 'VOYAGER 2 // PIA00062',
        wikiFile: 'Proteus_(Voyager_2).jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Proteus_(Voyager_2).jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00062/PIA00062~medium.jpg',
        nasaSearchQuery: 'Proteus Voyager 2 Neptune',
        fallbackText: 'PROTEUS+NEPTUNE+INNER+MOON',
        diameter: '420 km (Boxy Irregular)',
        distanceText: '117,647 km (From Neptune)',
        periodText: '26.9 Hours',
        tempText: '-222 °C (Mean)',
        eccentricityText: '0.0005',
        obliquityText: '0.00° (Locked)',
        description: 'Proteus is Neptune’s largest inner moon and the largest non-spherical body in the Solar System—sitting right at the mass threshold where self-gravity begins pulling an irregular polyhedron toward a sphere.',
        subFeatures: [
          {
            title: 'PHAROS IMPACT CRATER',
            detail: 'A 255 km crater spanning over half of Proteus’s diameter, excavated by an impact that nearly broke the moon apart.'
          },
          {
            title: 'SOOT-DARK ALBEDO (0.09)',
            detail: 'Proteus reflects less than 10% of incoming sunlight due to organic compounds darkened by cosmic radiation.'
          },
          {
            title: 'POST-TRITON RE-ACCRETION',
            detail: 'Formed from re-accreted debris after Triton’s retrograde capture disrupted Neptune’s original inner moon system.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- TRITON ---
      {
        id: 'triton',
        name: 'TRITON (NEPTUNE I)',
        shortName: 'TRITON',
        category: 'moon',
        parentId: 'neptune',
        sector: 'SECTOR 09.2 // RETROGRADE CAPTURED KUIPER WORLD',
        radius: 0.4,
        distance: 4.35,
        eccentricity: 0.000016,
        inclination: 2.74,
        ascendingNode: 2.29,
        argPeriapsis: 3.8,
        orbitSpeed: -0.036,
        tidallyLocked: true,
        tilt: 0.4943,
        oblateness: 0,
        bumpScale: 0.024,
        atmosphere: { dayColor: [0.78, 0.86, 1.0], sunsetColor: [0.95, 0.72, 0.72], density: 0.36, power: 4.4 },
        textureType: 'triton',
        nasaCatalog: 'VOYAGER 2 // PIA00317',
        wikiFile: 'Triton_moon_mosaic_Voyager_2_(large).jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Triton_moon_mosaic_Voyager_2_(large).jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA00317/PIA00317~medium.jpg',
        nasaSearchQuery: 'Triton moon mosaic Voyager 2',
        fallbackText: 'TRITON+NITROGEN+GEYSERS+VOYAGER',
        diameter: '2,707 km',
        distanceText: '354,759 km (From Neptune)',
        periodText: '5.877 Days (Retrograde)',
        tempText: '-235 °C (Coldest Measured Surface)',
        eccentricityText: '0.000016',
        obliquityText: '0.00° (Locked)',
        description: 'Triton is Neptune’s largest moon and the only large moon in the Solar System orbiting backward (retrograde) opposite its planet’s spin—proving it was captured from the Kuiper Belt. Active nitrogen geysers erupt across its pinkish-white nitrogen-ice south polar cap.',
        subFeatures: [
          {
            title: 'ACTIVE LIQUID NITROGEN GEYSERS',
            detail: 'Subsurface solar heating vaporizes trapped nitrogen ice, shooting 8 km-tall dark dust plumes that drift downwind across Triton’s south polar cap.'
          },
          {
            title: 'CANTALOUPE TERRAIN',
            detail: 'Dimpled, crisscrossing diapiric ridges resembling cantaloupe skin cover Triton’s western hemisphere.'
          },
          {
            title: 'RETROGRADE CAPTURE & TIDAL DECAY',
            detail: 'Orbiting backward at a 156.8° inclination, Triton’s orbit is slowly decaying toward Neptune’s Roche limit.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- PLUTO ---
      {
        id: 'pluto',
        name: 'PLUTO',
        shortName: 'PLUTO',
        category: 'planet',
        sector: 'SECTOR 10 // KUIPER BELT BINARY DWARF PLANET',
        radius: 0.56,
        distance: 168.0,
        eccentricity: 0.2488,
        inclination: 0.299,
        ascendingNode: 1.92,
        argPeriapsis: 1.98,
        orbitSpeed: 0.00095,
        rotationSpeed: -0.004,
        tilt: 2.138,
        oblateness: 0,
        bumpScale: 0.028,
        atmosphere: { dayColor: [0.65, 0.82, 1.0], sunsetColor: [0.85, 0.65, 0.55], density: 0.35, power: 4.5 },
        textureType: 'pluto',
        nasaCatalog: 'NEW HORIZONS // LORRI/MVIC',
        wikiFile: 'Pluto_in_True_Color_-_High-Res.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Pluto_in_True_Color_-_High-Res.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA19952/PIA19952~medium.jpg',
        nasaSearchQuery: 'Pluto New Horizons true color',
        fallbackText: 'PLUTO+TOMBAUGH+REGIO+HEART',
        diameter: '2,377 km',
        distanceText: '39.48 AU (5.91B km)',
        periodText: '247.94 Earth Years',
        tempText: '-229 °C (Mean Surface)',
        eccentricityText: '0.2488',
        obliquityText: '122.53° (Locked to Charon)',
        description: 'Pluto is the largest known dwarf planet by volume in the Kuiper Belt. Revealed in stunning detail by NASA’s New Horizons flyby, it features convective nitrogen glaciers, 3.5 km water-ice mountain ranges, and a blue-hazed nitrogen-methane atmosphere.',
        subFeatures: [
          {
            title: 'TOMBAUGH REGIO & SPUTNIK PLANITIA',
            detail: 'The bright heart-shaped region whose western lobe is a vast, crater-free convective glacier of solid nitrogen and carbon monoxide ice.'
          },
          {
            title: 'TENZING & HILLARY WATER-ICE MOUNTAINS',
            detail: 'Jagged blocks of water-ice bedrock standing 3.5 km high along the western border of Sputnik Planitia.'
          },
          {
            title: 'CTHULHU MACULA & BLUE HAZE',
            detail: 'Dark reddish tholin organic deposits contrast against bright nitrogen glaciers beneath 20 layered blue atmospheric haze strata.'
          }
        ],
        schematicType: 'icy-moon'
      },

      // --- CHARON ---
      {
        id: 'charon',
        name: 'CHARON (PLUTO I)',
        shortName: 'CHARON',
        category: 'moon',
        parentId: 'pluto',
        sector: 'SECTOR 10.1 // BINARY DWARF COMPANION',
        radius: 0.31,
        distance: 2.15,
        eccentricity: 0.0002,
        inclination: 2.138,
        ascendingNode: 1.92,
        argPeriapsis: 1.98,
        orbitSpeed: -0.025,
        tidallyLocked: true,
        tilt: 2.138,
        oblateness: 0,
        bumpScale: 0.03,
        textureType: 'charon',
        nasaCatalog: 'NEW HORIZONS // PIA19968',
        wikiFile: 'Charon_in_True_Color_-_High-Res.jpg',
        imageUrl: 'https://commons.wikimedia.org/wiki/Special:FilePath/Charon_in_True_Color_-_High-Res.jpg?width=800',
        backupImageUrl: 'https://images-assets.nasa.gov/image/PIA19968/PIA19968~medium.jpg',
        nasaSearchQuery: 'Charon New Horizons true color',
        fallbackText: 'CHARON+MORDOR+MACULA+SCAN',
        diameter: '1,212 km',
        distanceText: '19,591 km (From Pluto)',
        periodText: '6.387 Days (Mutual Lock)',
        tempText: '-220 °C (Mean)',
        eccentricityText: '0.0002',
        obliquityText: '0.00° (Locked)',
        description: 'Charon is the largest of Pluto’s five moons, measuring over half Pluto’s diameter. Because Charon is so massive relative to Pluto, the pair forms a true binary system orbiting a shared barycenter outside Pluto’s surface, with both worlds permanently keeping the same face locked toward each other.',
        subFeatures: [
          {
            title: 'MORDOR MACULA',
            detail: 'A vast dark reddish-brown polar cap near Charon’s north pole, composed of tholin macromolecules synthesized from methane gas escaping Pluto’s upper atmosphere.'
          },
          {
            title: 'SERENITY CHASMA',
            detail: 'An immense tectonic canyon belt stretching over 1,000 kilometers across Charon’s equator and plunging up to 7.5 kilometers deep, formed when a subsurface ocean froze and expanded.'
          },
          {
            title: 'MUTUAL TIDAL LOCK',
            detail: 'Pluto and Charon both rotate in exactly 6.387 Earth days, hanging motionless in each other’s sky as a double dwarf-planet system.'
          }
        ],
        schematicType: 'icy-moon'
      }
    ];

    // =========================================================================
    // 2. MULTI-STAGE SPACECRAFT IMAGE ACQUISITION & STRUCTURAL SVG GENERATOR
    // =========================================================================
    const resolvedImageCache = {};
    const inflightImagePromises = {};

    function probeImageLoad(url, timeoutMs = 6500) {
      return new Promise((resolve, reject) => {
        if (!url) {
          reject(new Error('Empty URL'));
          return;
        }
        const img = new Image();
        let settled = false;
        const timer = setTimeout(() => {
          if (!settled) {
            settled = true;
            img.src = '';
            reject(new Error('Image probe timeout'));
          }
        }, timeoutMs);

        img.onload = () => {
          if (!settled) {
            settled = true;
            clearTimeout(timer);
            if (img.naturalWidth > 16) {
              resolve(url);
            } else {
              reject(new Error('Invalid image dimensions'));
            }
          }
        };

        img.onerror = () => {
          if (!settled) {
            settled = true;
            clearTimeout(timer);
            reject(new Error('Image load failed'));
          }
        };

        img.src = url;
      });
    }

    function raceFirstValidPromise(promises) {
      return new Promise((resolve, reject) => {
        let pending = promises.length;
        if (pending === 0) {
          reject(new Error('No candidates'));
          return;
        }
        promises.forEach((p) => {
          Promise.resolve(p).then(
            (val) => resolve(val),
            () => {
              pending -= 1;
              if (pending === 0) reject(new Error('All image sources failed'));
            }
          );
        });
      });
    }

    async function fetchWikimediaPhotoUrl(body) {
      if (body.wikiFile) {
        try {
          const titleParam = encodeURIComponent(`File:${body.wikiFile}`);
          const apiUrl = `https://commons.wikimedia.org/w/api.php?action=query&titles=${titleParam}&prop=imageinfo&iiprop=url&iiurlwidth=800&format=json&origin=*`;
          const resp = await fetch(apiUrl);
          if (resp.ok) {
            const data = await resp.json();
            const pages = (data && data.query && data.query.pages) || {};
            const pageKeys = Object.keys(pages);
            for (let i = 0; i < pageKeys.length; i++) {
              const info = pages[pageKeys[i]] && pages[pageKeys[i]].imageinfo && pages[pageKeys[i]].imageinfo[0];
              const candidate = info && (info.thumburl || info.url);
              if (candidate) {
                return await probeImageLoad(candidate, 7000);
              }
            }
          }
        } catch (e) {
          // Continue to Wikimedia search fallback
        }
      }

      const searchQ = encodeURIComponent(`${body.nasaSearchQuery || body.shortName} filetype:bitmap`);
      const searchApi = `https://commons.wikimedia.org/w/api.php?action=query&generator=search&gsrsearch=${searchQ}&gsrnamespace=6&gsrlimit=4&prop=imageinfo&iiprop=url&iiurlwidth=800&format=json&origin=*`;
      const sResp = await fetch(searchApi);
      if (!sResp.ok) throw new Error('Wikimedia search failed');
      const sData = await sResp.json();
      const sPages = (sData && sData.query && sData.query.pages) || {};
      const keys = Object.keys(sPages);
      for (let i = 0; i < keys.length; i++) {
        const info = sPages[keys[i]] && sPages[keys[i]].imageinfo && sPages[keys[i]].imageinfo[0];
        const candidate = info && (info.thumburl || info.url);
        if (candidate && !candidate.toLowerCase().endsWith('.svg')) {
          try {
            return await probeImageLoad(candidate, 6500);
          } catch (err) {
            // Try next candidate
          }
        }
      }
      throw new Error('No valid Wikimedia bitmap found');
    }

    async function fetchNasaLibraryPhotoUrl(body) {
      const q = encodeURIComponent(body.nasaSearchQuery || body.name);
      const resp = await fetch(`https://images-api.nasa.gov/search?q=${q}&media_type=image`);
      if (!resp.ok) throw new Error('NASA API unavailable');
      const json = await resp.json();
      const items = (json && json.collection && json.collection.items) || [];
      for (let i = 0; i < Math.min(items.length, 5); i++) {
        const links = items[i].links || [];
        const imgLink = links.find((l) => l.render === 'image' && l.href);
        if (imgLink && imgLink.href) {
          try {
            return await probeImageLoad(imgLink.href, 6500);
          } catch (err) {
            // Try next NASA result
          }
        }
      }
      throw new Error('No valid NASA library photo found');
    }

    function createOfflineOrbitalCaptureUrl(body) {
      return new Promise((resolve) => {
        try {
          const snapCanvas = document.createElement('canvas');
          snapCanvas.width = 640;
          snapCanvas.height = 360;
          const ctx = snapCanvas.getContext('2d');
          ctx.fillStyle = '#000000';
          ctx.fillRect(0, 0, 640, 360);

          for (let i = 0; i < 90; i++) {
            const sx = (Math.sin(i * 91.7) * 0.5 + 0.5) * 640;
            const sy = (Math.cos(i * 47.3) * 0.5 + 0.5) * 360;
            ctx.fillStyle = i % 5 === 0 ? 'rgba(255,0,0,0.6)' : 'rgba(255,255,255,0.55)';
            ctx.fillRect(sx, sy, 1.5, 1.5);
          }

          const entity = entityById[body.id];
          const texImg = entity && entity.mesh && entity.mesh.material && entity.mesh.material.map && entity.mesh.material.map.image;
          const cx = 320;
          const cy = 180;
          const r = body.hasRings ? 84 : 112;

          if (body.hasRings) {
            ctx.save();
            ctx.translate(cx, cy);
            ctx.rotate(-0.32);
            ctx.scale(1, 0.34);
            ctx.strokeStyle = 'rgba(225, 205, 165, 0.78)';
            ctx.lineWidth = 22;
            ctx.beginPath();
            ctx.arc(0, 0, 155, 0, Math.PI * 2);
            ctx.stroke();
            ctx.restore();
          }

          ctx.save();
          ctx.beginPath();
          ctx.arc(cx, cy, r, 0, Math.PI * 2);
          ctx.clip();
          if (texImg) {
            ctx.drawImage(texImg, cx - r * 1.4, cy - r, r * 2.8, r * 2);
          } else {
            ctx.fillStyle = '#888888';
            ctx.fillRect(cx - r, cy - r, r * 2, r * 2);
          }
          if (body.id !== 'sun') {
            const shade = ctx.createRadialGradient(cx - r * 0.35, cy - r * 0.25, r * 0.15, cx, cy, r * 1.08);
            shade.addColorStop(0, 'rgba(0,0,0,0)');
            shade.addColorStop(0.68, 'rgba(0,0,0,0.28)');
            shade.addColorStop(1, 'rgba(0,0,0,0.92)');
            ctx.fillStyle = shade;
            ctx.fillRect(cx - r, cy - r, r * 2, r * 2);
          }
          ctx.restore();

          snapCanvas.toBlob((blob) => {
            if (blob) {
              resolve(URL.createObjectURL(blob));
            } else {
              resolve(`https://placehold.co/800x440/000000/ff0000?text=${body.fallbackText}`);
            }
          }, 'image/png');
        } catch (err) {
          resolve(`https://placehold.co/800x440/000000/ff0000?text=${body.fallbackText}`);
        }
      });
    }

    function resolveRealSpacecraftUrl(body) {
      if (resolvedImageCache[body.id]) {
        return Promise.resolve(resolvedImageCache[body.id]);
      }
      if (inflightImagePromises[body.id]) {
        return inflightImagePromises[body.id];
      }

      const probes = [
        probeImageLoad(body.imageUrl, 6000),
        probeImageLoad(body.backupImageUrl, 6000),
        fetchWikimediaPhotoUrl(body),
        fetchNasaLibraryPhotoUrl(body)
      ];

      inflightImagePromises[body.id] = raceFirstValidPromise(probes)
        .catch(() => createOfflineOrbitalCaptureUrl(body))
        .then((finalUrl) => {
          resolvedImageCache[body.id] = finalUrl;
          delete inflightImagePromises[body.id];
          return finalUrl;
        });

      return inflightImagePromises[body.id];
    }

    function prewarmAllSpacecraftImages() {
      let idx = 0;
      const batchSize = 4;
      function nextBatch() {
        const end = Math.min(idx + batchSize, SOLAR_BODIES.length);
        for (; idx < end; idx++) {
          resolveRealSpacecraftUrl(SOLAR_BODIES[idx]);
        }
        if (idx < SOLAR_BODIES.length) {
          setTimeout(nextBatch, 450);
        }
      }
      setTimeout(nextBatch, 600);
    }

    /**
     * Generates a high-tech #000000 / #ff0000 / #ffffff internal structural SVG schematic.
     */
    function generateStructuralSVG(body) {
      const layerSets = {
        star: [
          { r: 72, stroke: '#ff0000', fill: 'rgba(255,0,0,0.08)', label: 'CORONA / PHOTOSPHERE' },
          { r: 52, stroke: '#ffffff', fill: 'rgba(255,255,255,0.05)', label: 'CONVECTIVE ZONE' },
          { r: 34, stroke: '#ff0000', fill: 'rgba(255,0,0,0.14)', label: 'RADIATIVE ZONE' },
          { r: 16, stroke: '#ffffff', fill: '#ff0000', label: '15.7M K FUSION CORE' }
        ],
        terrestrial: [
          { r: 72, stroke: '#ffffff', fill: 'rgba(255,255,255,0.04)', label: 'SILICATE CRUST & ATMOSPHERE' },
          { r: 54, stroke: '#ff0000', fill: 'rgba(255,0,0,0.08)', label: 'SILICATE MANTLE' },
          { r: 32, stroke: '#ffffff', fill: 'rgba(255,255,255,0.08)', label: 'LIQUID OUTER CORE' },
          { r: 15, stroke: '#ff0000', fill: '#ff0000', label: 'SOLID IRON-NICKEL CORE' }
        ],
        'gas-giant': [
          { r: 72, stroke: '#ffffff', fill: 'rgba(255,255,255,0.04)', label: 'H₂ / He CLOUD DECK' },
          { r: 54, stroke: '#ff0000', fill: 'rgba(255,0,0,0.08)', label: 'LIQUID MOLECULAR H₂' },
          { r: 34, stroke: '#ffffff', fill: 'rgba(255,255,255,0.09)', label: 'METALLIC HYDROGEN' },
          { r: 14, stroke: '#ff0000', fill: '#ff0000', label: 'DENSE ROCK-ICE CORE' }
        ],
        'ice-giant': [
          { r: 72, stroke: '#ffffff', fill: 'rgba(255,255,255,0.04)', label: 'H₂ / He / CH₄ ATMOSPHERE' },
          { r: 52, stroke: '#ff0000', fill: 'rgba(255,0,0,0.1)', label: 'SUPERIONIC H₂O-NH₃-CH₄' },
          { r: 22, stroke: '#ffffff', fill: '#ff0000', label: 'SILICATE-IRON CORE' }
        ],
        ringed: [
          { r: 62, stroke: '#ffffff', fill: 'rgba(255,255,255,0.04)', label: 'H₂ / He ZONAL CLOUDS' },
          { r: 44, stroke: '#ff0000', fill: 'rgba(255,0,0,0.09)', label: 'METALLIC HYDROGEN' },
          { r: 16, stroke: '#ffffff', fill: '#ff0000', label: 'ROCK-ICE CORE' }
        ],
        'icy-moon': [
          { r: 72, stroke: '#ffffff', fill: 'rgba(255,255,255,0.05)', label: 'WATER-ICE / REGOLITH CRUST' },
          { r: 50, stroke: '#ff0000', fill: 'rgba(255,0,0,0.09)', label: 'SUBSURFACE OCEAN / ICE MANTLE' },
          { r: 24, stroke: '#ffffff', fill: '#ff0000', label: 'HYDRATED SILICATE CORE' }
        ]
      };

      const layers = layerSets[body.schematicType] || layerSets['icy-moon'];
      const ringsExtra = body.schematicType === 'ringed'
        ? `<ellipse cx="105" cy="85" rx="96" ry="22" fill="none" stroke="#ff0000" stroke-width="2" stroke-dasharray="6 3" />
           <ellipse cx="105" cy="85" rx="82" ry="18" fill="none" stroke="#ffffff" stroke-width="1.2" />`
        : '';

      const circlesSvg = layers.map((l) => `
        <circle cx="105" cy="85" r="${l.r}" fill="${l.fill}" stroke="${l.stroke}" stroke-width="1.5" />
      `).join('');

      const calloutsSvg = layers.map((l, i) => {
        const yPos = 24 + i * 36;
        return `
          <line x1="${105 + l.r * 0.65}" y1="${85 - l.r * 0.35}" x2="205" y2="${yPos}" stroke="#ff0000" stroke-width="1" />
          <circle cx="205" cy="${yPos}" r="2.5" fill="#ff0000" />
          <text x="212" y="${yPos + 3}" fill="#ffffff" font-family="JetBrains Mono, monospace" font-size="8.5" font-weight="700">${l.label}</text>
        `;
      }).join('');

      return `
        <svg viewBox="0 0 380 170" class="w-full h-full">
          <defs>
            <pattern id="techGrid" width="16" height="16" patternUnits="userSpaceOnUse">
              <path d="M 16 0 L 0 0 0 16" fill="none" stroke="rgba(255,255,255,0.06)" stroke-width="0.6"/>
            </pattern>
          </defs>
          <rect width="380" height="170" fill="url(#techGrid)" />
          ${ringsExtra}
          ${circlesSvg}
          <line x1="105" y1="8" x2="105" y2="162" stroke="rgba(255,0,0,0.45)" stroke-width="1" stroke-dasharray="3 3" />
          <line x1="25" y1="85" x2="185" y2="85" stroke="rgba(255,0,0,0.45)" stroke-width="1" stroke-dasharray="3 3" />
          ${calloutsSvg}
        </svg>
      `;
    }

    // =========================================================================
    // 3. THREE.JS SCENE, CAMERA, LIGHTING, STARFIELD & AUDIO SYNTHESIZER
    // =========================================================================
    const canvasWrapper = document.getElementById('canvasWrapper');
    const canvas = document.getElementById('solarCanvas');

    const renderer = new THREE.WebGLRenderer({
      canvas,
      antialias: true,
      powerPreference: 'high-performance'
    });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
    renderer.setSize(canvasWrapper.clientWidth, canvasWrapper.clientHeight);
    renderer.setClearColor(0x000000, 1);

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x000000);

    const camera = new THREE.PerspectiveCamera(
      46,
      canvasWrapper.clientWidth / canvasWrapper.clientHeight,
      0.08,
      2000
    );

    // Solar Point Light + Subtle Ambient Fill
    const sunLight = new THREE.PointLight(0xffffff, 2.35, 950, 0.35);
    sunLight.position.set(0, 0, 0);
    scene.add(sunLight);

    const ambientLight = new THREE.AmbientLight(0x283244, 0.65);
    scene.add(ambientLight);

    // Build Multi-Spectral Background Starfield
    function createStarField() {
      const starCount = 3600;
      const positions = new Float32Array(starCount * 3);
      const colors = new Float32Array(starCount * 3);

      for (let i = 0; i < starCount; i++) {
        const theta = Math.random() * Math.PI * 2;
        const phi = Math.acos(2 * Math.random() - 1);
        const r = 420 + Math.random() * 520;

        positions[i * 3] = r * Math.sin(phi) * Math.cos(theta);
        positions[i * 3 + 1] = r * Math.cos(phi);
        positions[i * 3 + 2] = r * Math.sin(phi) * Math.sin(theta);

        const tint = Math.random();
        if (tint < 0.14) {
          colors[i * 3] = 0.78;
          colors[i * 3 + 1] = 0.88;
          colors[i * 3 + 2] = 1.0;
        } else if (tint < 0.28) {
          colors[i * 3] = 1.0;
          colors[i * 3 + 1] = 0.86;
          colors[i * 3 + 2] = 0.72;
        } else {
          const b = 0.55 + Math.random() * 0.45;
          colors[i * 3] = b;
          colors[i * 3 + 1] = b;
          colors[i * 3 + 2] = b;
        }
      }

      const geo = new THREE.BufferGeometry();
      geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      geo.setAttribute('color', new THREE.BufferAttribute(colors, 3));

      const mat = new THREE.PointsMaterial({
        size: 1.35,
        vertexColors: true,
        transparent: true,
        opacity: 0.9,
        sizeAttenuation: true
      });

      const stars = new THREE.Points(geo, mat);
      scene.add(stars);
      return stars;
    }

    const starField = createStarField();

    function createSunGlowTexture() {
      const c = document.createElement('canvas');
      c.width = 256;
      c.height = 256;
      const ctx = c.getContext('2d');
      const grad = ctx.createRadialGradient(128, 128, 12, 128, 128, 128);
      grad.addColorStop(0.0, 'rgba(255, 245, 210, 1.0)');
      grad.addColorStop(0.22, 'rgba(255, 175, 55, 0.78)');
      grad.addColorStop(0.55, 'rgba(255, 65, 15, 0.25)');
      grad.addColorStop(1.0, 'rgba(0, 0, 0, 0.0)');
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, 256, 256);
      return new THREE.CanvasTexture(c);
    }

    // Simulation State & Web Audio Synthesizer
    let simulationSpeed = 1.0;
    let showOrbits = true;
    let audioEnabled = true;
    let audioCtx = null;
    let selectedIndex = 0;
    let activeRosterFilter = 'all';

    function playHudTone(freq = 640, duration = 0.06) {
      if (!audioEnabled) return;
      try {
        if (!audioCtx) {
          const AudioCtx = window.AudioContext || window.webkitAudioContext;
          if (AudioCtx) audioCtx = new AudioCtx();
        }
        if (!audioCtx) return;
        if (audioCtx.state === 'suspended') audioCtx.resume();

        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(freq * 1.35, audioCtx.currentTime + duration);

        gain.gain.setValueAtTime(0.045, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);

        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + duration);
      } catch (e) {
        // Ignore browser autoplay restrictions
      }
    }

    // =========================================================================
    // 4. HYPER-REALISTIC 3D SPHERICAL TOPOGRAPHY & EARTH BLUE MARBLE ENGINE
    // =========================================================================
    function hash3(x, y, z) {
      const h = Math.sin(x * 127.1 + y * 311.7 + z * 74.7) * 43758.5453123;
      return h - Math.floor(h);
    }

    function noise3D(x, y, z) {
      const ix = Math.floor(x);
      const iy = Math.floor(y);
      const iz = Math.floor(z);
      const fx = x - ix;
      const fy = y - iy;
      const fz = z - iz;
      const ux = fx * fx * (3 - 2 * fx);
      const uy = fy * fy * (3 - 2 * fy);
      const uz = fz * fz * (3 - 2 * fz);

      const n000 = hash3(ix, iy, iz);
      const n100 = hash3(ix + 1, iy, iz);
      const n010 = hash3(ix, iy + 1, iz);
      const n110 = hash3(ix + 1, iy + 1, iz);
      const n001 = hash3(ix, iy, iz + 1);
      const n101 = hash3(ix + 1, iy, iz + 1);
      const n011 = hash3(ix, iy + 1, iz + 1);
      const n111 = hash3(ix + 1, iy + 1, iz + 1);

      const nx00 = n000 + (n100 - n000) * ux;
      const nx10 = n010 + (n110 - n010) * ux;
      const nx01 = n001 + (n101 - n001) * ux;
      const nx11 = n011 + (n111 - n011) * ux;

      const nxy0 = nx00 + (nx10 - nx00) * uy;
      const nxy1 = nx01 + (nx11 - nx01) * uy;

      return nxy0 + (nxy1 - nxy0) * uz;
    }

    function fbm3D(x, y, z, octaves = 4) {
      let val = 0;
      let amp = 0.5;
      let freq = 1.0;
      for (let i = 0; i < octaves; i++) {
        val += amp * noise3D(x * freq, y * freq, z * freq);
        freq *= 2.04;
        amp *= 0.5;
      }
      return val;
    }

    function createIrregularGeometry(baseRadius, seed = 1.0) {
      const geo = new THREE.SphereGeometry(baseRadius, 36, 24);
      const pos = geo.attributes.position;
      for (let i = 0; i < pos.count; i++) {
        const vx = pos.getX(i);
        const vy = pos.getY(i);
        const vz = pos.getZ(i);
        const len = Math.sqrt(vx * vx + vy * vy + vz * vz) || 1;
        const nx = vx / len;
        const ny = vy / len;
        const nz = vz / len;

        const stretch = 1.0 + 0.24 * nx * nx - 0.16 * ny * ny + 0.12 * nz;
        const lump = 0.88 + 0.24 * fbm3D(nx * 2.5 + seed, ny * 2.5, nz * 2.5, 3);
        const r = baseRadius * stretch * lump;
        pos.setXYZ(i, nx * r, ny * r, nz * r);
      }
      geo.computeVertexNormals();
      return geo;
    }

    const lunarMaria = [
      { x: -0.22, y: 0.52, z: 0.82, r: 0.34 }, // Mare Imbrium
      { x: -0.62, y: 0.24, z: 0.74, r: 0.42 }, // Oceanus Procellarum
      { x: 0.24, y: 0.44, z: 0.86, r: 0.24 },  // Mare Serenitatis
      { x: 0.46, y: 0.18, z: 0.87, r: 0.26 },  // Mare Tranquillitatis
      { x: 0.76, y: 0.28, z: 0.58, r: 0.18 },  // Mare Crisium
      { x: 0.62, y: -0.14, z: 0.77, r: 0.22 }, // Mare Foecunditatis
      { x: -0.26, y: -0.34, z: 0.90, r: 0.24 },// Mare Nubium
      { x: -0.54, y: -0.38, z: 0.75, r: 0.19 },// Mare Humorum
      { x: 0.05, y: 0.78, z: 0.62, r: 0.24 }   // Mare Frigoris
    ];

    /**
     * Generates high-resolution 1024x512 cartographic Blue Marble Earth maps:
     * continents, mountain ranges, coral shelves, dynamic clouds, and night city lights.
     */
    function generateEarthMaps() {
      const W = 1024;
      const H = 512;

      const albedoCanvas = document.createElement('canvas');
      albedoCanvas.width = W;
      albedoCanvas.height = H;
      const aCtx = albedoCanvas.getContext('2d');
      const aImg = aCtx.createImageData(W, H);

      const bumpCanvas = document.createElement('canvas');
      bumpCanvas.width = W;
      bumpCanvas.height = H;
      const bCtx = bumpCanvas.getContext('2d');
      const bImg = bCtx.createImageData(W, H);

      const roughCanvas = document.createElement('canvas');
      roughCanvas.width = W;
      roughCanvas.height = H;
      const rCtx = roughCanvas.getContext('2d');
      const rImg = rCtx.createImageData(W, H);

      const cloudCanvas = document.createElement('canvas');
      cloudCanvas.width = W;
      cloudCanvas.height = H;
      const cCtx = cloudCanvas.getContext('2d');
      const cImg = cCtx.createImageData(W, H);

      const nightCanvas = document.createElement('canvas');
      nightCanvas.width = W;
      nightCanvas.height = H;
      const nCtx = nightCanvas.getContext('2d');
      const nImg = nCtx.createImageData(W, H);

      // Geographic continental Gaussian blobs [lonDeg, latDeg, radLon, radLat, weight]
      const landmasses = [
        // North America
        [-105, 48, 26, 15, 1.05], [-118, 56, 16, 11, 0.95], [-84, 52, 16, 11, 0.9],
        [-148, 64, 14, 7, 0.85], [-98, 36, 18, 11, 1.0], [-102, 22, 9, 8, 0.82],
        [-82, 28, 4.5, 5.5, 0.72], [-72, 44, 9, 7, 0.82],
        // Greenland
        [-42, 72, 13, 8, 1.05],
        // South America
        [-64, -8, 15, 13, 1.08], [-54, -18, 13, 12, 1.0], [-66, -34, 8, 14, 0.95],
        [-70, -48, 5, 9, 0.82],
        // Europe
        [14, 50, 18, 10, 0.95], [26, 61, 14, 9, 0.88], [-4, 41, 6, 5, 0.82],
        [14, 42, 4, 5, 0.74], [-2, 54, 4, 4.5, 0.74],
        // Africa
        [18, 16, 22, 14, 1.12], [-4, 14, 14, 9, 1.0], [34, 4, 12, 13, 1.02],
        [24, -18, 11, 14, 1.02], [46, -19, 3.8, 7, 0.82],
        // Asia & Middle East
        [46, 25, 11, 9, 0.95], [68, 56, 28, 14, 1.12], [105, 58, 30, 13, 1.1],
        [142, 62, 16, 8, 0.92], [96, 36, 22, 13, 1.15], [112, 30, 14, 11, 1.02],
        [78, 20, 8, 10, 0.98], [102, 15, 7, 9, 0.86], [138, 36, 4.2, 6.5, 0.76],
        // Maritime Southeast Asia
        [102, -2, 6, 4.5, 0.75], [114, 1, 6, 5, 0.82], [122, 12, 4, 5, 0.72],
        [140, -5, 8, 4.5, 0.8],
        // Australia & New Zealand
        [134, -25, 16, 10, 1.06], [172, -42, 4.5, 5.5, 0.76],
        // Antarctica
        [0, -82, 180, 12, 1.25]
      ];

      // Major Urban Night-Light Corridors [lonDeg, latDeg, radiusDeg]
      const cityHubs = [
        [-77, 40, 9], [-87, 41, 8], [-118, 35, 6], [-96, 31, 6], [-99, 19, 5],
        [-46, -23, 7], [-58, -34, 6],
        [2, 49, 11], [12, 45, 9], [37, 55, 8], [-3, 40, 5],
        [31, 30, 6], [4, 7, 6], [28, -26, 5], [52, 26, 6],
        [77, 22, 12], [86, 25, 9], [116, 36, 12], [121, 31, 10], [114, 23, 7],
        [139, 36, 6], [127, 37, 5], [104, 13, 6], [107, -6, 6], [151, -33, 5]
      ];

      const sinPhiArr = new Float32Array(H);
      const cosPhiArr = new Float32Array(H);
      const latDegArr = new Float32Array(H);
      for (let y = 0; y < H; y++) {
        const v = y / (H - 1);
        const phi = v * Math.PI;
        sinPhiArr[y] = Math.sin(phi);
        cosPhiArr[y] = Math.cos(phi);
        latDegArr[y] = 90 - v * 180;
      }

      const sinThetaArr = new Float32Array(W);
      const cosThetaArr = new Float32Array(W);
      const lonDegArr = new Float32Array(W);
      for (let x = 0; x < W; x++) {
        const u = x / W;
        const theta = u * Math.PI * 2;
        sinThetaArr[x] = Math.sin(theta);
        cosThetaArr[x] = Math.cos(theta);
        lonDegArr[x] = u * 360 - 180;
      }

      for (let y = 0; y < H; y++) {
        const sinP = sinPhiArr[y];
        const ny = cosPhiArr[y];
        const lat = latDegArr[y];
        const absLat = Math.abs(lat);

        for (let x = 0; x < W; x++) {
          const nx = -sinP * cosThetaArr[x];
          const nz = sinP * sinThetaArr[x];
          const lon = lonDegArr[x];

          // Evaluate continental mask
          let landScore = 0;
          for (let i = 0; i < landmasses.length; i++) {
            const lm = landmasses[i];
            let dLon = Math.abs(lon - lm[0]);
            if (dLon > 180) dLon = 360 - dLon;
            const dLat = lat - lm[1];
            const distSq = (dLon * dLon) / (lm[2] * lm[2]) + (dLat * dLat) / (lm[3] * lm[3]);
            if (distSq < 1.35) {
              const contrib = lm[4] * Math.max(0, 1 - distSq * 0.78);
              if (contrib > landScore) landScore = contrib;
            }
          }

          const coastDetail = fbm3D(nx * 5.2 + 1.3, ny * 5.2 - 0.7, nz * 5.2 + 2.1, 4);
          const fineDetail = fbm3D(nx * 16.0, ny * 16.0, nz * 16.0, 3);
          const elevation = landScore + (coastDetail - 0.48) * 0.62 + (fineDetail - 0.5) * 0.16;

          const idx = (y * W + x) * 4;
          let r = 0;
          let g = 0;
          let b = 0;
          let bumpVal = 128;
          let roughVal = 210;

          if (elevation < 0.44) {
            // Ocean + shallow turquoise coral shelves near coastlines
            const shelf = Math.max(0, (elevation - 0.34) / 0.1);
            const oceanVar = (fineDetail - 0.5) * 10;
            r = Math.max(6, Math.min(48, 8 + shelf * 26 + oceanVar * 0.5));
            g = Math.max(24, Math.min(148, 34 + shelf * 92 + oceanVar));
            b = Math.max(78, Math.min(198, 98 + shelf * 78 + oceanVar));
            bumpVal = 118;
            roughVal = 42; // Specular ocean sun glint
          } else {
            // Land biomes: tropical forest, temperate green, arid desert, alpine snow
            const hNorm = Math.min(1, (elevation - 0.44) / 0.55);
            const moisture = fbm3D(nx * 3.8 + 4.1, ny * 3.8 + 1.9, nz * 3.8 - 2.4, 3);
            const desertBelt = Math.exp(-Math.pow((absLat - 23) / 11, 2)) * (1 - moisture * 0.95);

            // Lush forest base
            let lr = 36 + fineDetail * 28;
            let lg = 84 + fineDetail * 42;
            let lb = 32 + fineDetail * 18;

            // Blend arid desert ochre (Sahara, Arabia, Outback, Atacama)
            const arid = Math.max(0, Math.min(1, desertBelt * 1.45));
            lr = lr * (1 - arid) + (186 + fineDetail * 32) * arid;
            lg = lg * (1 - arid) + (152 + fineDetail * 28) * arid;
            lb = lb * (1 - arid) + (98 + fineDetail * 22) * arid;

            // Blend high mountain rock & snow
            const snowFactor = Math.max(
              0,
              Math.min(1, (hNorm - 0.56) * 2.8 + Math.max(0, (absLat - 63) / 14))
            );
            r = lr * (1 - snowFactor) + 242 * snowFactor;
            g = lg * (1 - snowFactor) + 246 * snowFactor;
            b = lb * (1 - snowFactor) + 252 * snowFactor;

            bumpVal = Math.min(255, Math.floor(132 + hNorm * 120));
            roughVal = 228;
          }

          // Polar sea-ice caps
          if (absLat > 75 + (coastDetail - 0.5) * 9) {
            r = 240;
            g = 246;
            b = 252;
            bumpVal = 142;
            roughVal = 180;
          }

          aImg.data[idx] = r;
          aImg.data[idx + 1] = g;
          aImg.data[idx + 2] = b;
          aImg.data[idx + 3] = 255;

          bImg.data[idx] = bumpVal;
          bImg.data[idx + 1] = bumpVal;
          bImg.data[idx + 2] = bumpVal;
          bImg.data[idx + 3] = 255;

          rImg.data[idx] = roughVal;
          rImg.data[idx + 1] = roughVal;
          rImg.data[idx + 2] = roughVal;
          rImg.data[idx + 3] = 255;

          // Swirling weather cloud bands
          const swirl = fbm3D(nx * 2.4 + 3.1, ny * 2.4, nz * 2.4 - 1.4, 3);
          const cloudNoise = fbm3D(
            nx * 4.8 + swirl * 1.4,
            ny * 6.5,
            nz * 4.8 - swirl * 1.4,
            4
          );
          const cloudAlpha = Math.max(0, Math.min(235, (cloudNoise - 0.52) * 620));
          cImg.data[idx] = 255;
          cImg.data[idx + 1] = 255;
          cImg.data[idx + 2] = 255;
          cImg.data[idx + 3] = cloudAlpha;

          // Night-side city lights on land
          let cityIntensity = 0;
          if (elevation >= 0.44 && absLat < 66) {
            for (let c = 0; c < cityHubs.length; c++) {
              const hub = cityHubs[c];
              let dLon = Math.abs(lon - hub[0]);
              if (dLon > 180) dLon = 360 - dLon;
              const dLat = lat - hub[1];
              const dSq = (dLon * dLon + dLat * dLat) / (hub[2] * hub[2]);
              if (dSq < 1.0) {
                const spark = noise3D(nx * 48, ny * 48, nz * 48);
                if (spark > 0.54) {
                  const val = (1 - dSq) * ((spark - 0.54) / 0.46);
                  if (val > cityIntensity) cityIntensity = val;
                }
              }
            }
          }
          nImg.data[idx] = Math.min(255, Math.floor(cityIntensity * 255));
          nImg.data[idx + 1] = Math.min(255, Math.floor(cityIntensity * 190));
          nImg.data[idx + 2] = Math.min(255, Math.floor(cityIntensity * 85));
          nImg.data[idx + 3] = 255;
        }
      }

      aCtx.putImageData(aImg, 0, 0);
      bCtx.putImageData(bImg, 0, 0);
      rCtx.putImageData(rImg, 0, 0);
      cCtx.putImageData(cImg, 0, 0);
      nCtx.putImageData(nImg, 0, 0);

      const mapTex = new THREE.CanvasTexture(albedoCanvas);
      const bumpTex = new THREE.CanvasTexture(bumpCanvas);
      const roughTex = new THREE.CanvasTexture(roughCanvas);
      const cloudTex = new THREE.CanvasTexture(cloudCanvas);
      const nightTex = new THREE.CanvasTexture(nightCanvas);

      [mapTex, bumpTex, roughTex, cloudTex, nightTex].forEach((t) => {
        t.wrapS = THREE.RepeatWrapping;
        t.wrapT = THREE.ClampToEdgeWrapping;
      });

      return { map: mapTex, bumpMap: bumpTex, roughnessMap: roughTex, cloudMap: cloudTex, nightMap: nightTex };
    }

    /**
     * Synthesizes seamless 3D spherical Albedo, Bump, and Roughness textures for every body.
     */
    function generateBodyMaps(type) {
      if (type === 'earth') {
        return generateEarthMaps();
      }

      const W = 512;
      const H = 256;

      const albedoCanvas = document.createElement('canvas');
      albedoCanvas.width = W;
      albedoCanvas.height = H;
      const aCtx = albedoCanvas.getContext('2d');
      const aImg = aCtx.createImageData(W, H);

      const bumpCanvas = document.createElement('canvas');
      bumpCanvas.width = W;
      bumpCanvas.height = H;
      const bCtx = bumpCanvas.getContext('2d');
      const bImg = bCtx.createImageData(W, H);

      const roughCanvas = document.createElement('canvas');
      roughCanvas.width = W;
      roughCanvas.height = H;
      const rCtx = roughCanvas.getContext('2d');
      const rImg = rCtx.createImageData(W, H);

      for (let y = 0; y < H; y++) {
        const v = y / (H - 1);
        const phi = v * Math.PI;
        const sinP = Math.sin(phi);
        const ny = Math.cos(phi);
        const lat = 90 - v * 180;
        const absLat = Math.abs(lat);

        for (let x = 0; x < W; x++) {
          const u = x / W;
          const theta = u * Math.PI * 2;
          const nx = -sinP * Math.cos(theta);
          const nz = sinP * Math.sin(theta);
          const lon = u * 360 - 180;

          const nLow = fbm3D(nx * 2.6 + 1.1, ny * 2.6 - 0.5, nz * 2.6 + 0.8, 3);
          const nMid = fbm3D(nx * 6.5 - 2.1, ny * 6.5 + 1.4, nz * 6.5 - 0.9, 4);
          const nHigh = noise3D(nx * 18.0, ny * 18.0, nz * 18.0);

          let r = 160;
          let g = 160;
          let b = 160;
          let bump = 128;
          let rough = 225;

          if (type === 'sun') {
            const gran = fbm3D(nx * 9.0, ny * 9.0, nz * 9.0, 4);
            const flare = Math.pow(nMid, 1.5);
            r = 255;
            g = Math.min(255, Math.floor(135 + gran * 95 + flare * 35));
            b = Math.min(255, Math.floor(24 + gran * 75 + flare * 40));
          } else if (type === 'mercury' || type === 'phobos' || type === 'dark-rock') {
            const base = type === 'dark-rock' ? 68 : type === 'phobos' ? 92 : 134;
            const cField = Math.sin(nMid * 22.0) * 0.5 + 0.5;
            const shade = base + (nLow - 0.5) * 44 + (cField - 0.5) * 28 + (nHigh - 0.5) * 14;
            r = shade * 1.03;
            g = shade * 0.99;
            b = shade * 0.94;
            bump = Math.max(0, Math.min(255, Math.floor(128 + (cField - 0.5) * 90 + (nHigh - 0.5) * 35)));
          } else if (type === 'tan-rock') {
            const shade = 148 + (nLow - 0.5) * 36 + (nMid - 0.5) * 22;
            r = shade * 1.08;
            g = shade * 0.98;
            b = shade * 0.84;
            bump = Math.floor(128 + (nMid - 0.5) * 60);
          } else if (type === 'venus') {
            const swirl = fbm3D(nx * 2.2 + nLow * 1.6, ny * 4.5, nz * 2.2 - nLow * 1.6, 4);
            r = 226 + (swirl - 0.5) * 38;
            g = 184 + (swirl - 0.5) * 44;
            b = 112 + (swirl - 0.5) * 36;
            bump = Math.floor(128 + (swirl - 0.5) * 25);
            rough = 175;
          } else if (type === 'luna') {
            let mariaMask = 0;
            for (let m = 0; m < lunarMaria.length; m++) {
              const lm = lunarMaria[m];
              const dx = nx - lm.x;
              const dy = ny - lm.y;
              const dz = nz - lm.z;
              const dist = Math.sqrt(dx * dx + dy * dy + dz * dz);
              if (dist < lm.r) {
                const factor = 1 - dist / lm.r;
                if (factor > mariaMask) mariaMask = factor;
              }
            }
            const cRing = Math.sin(nMid * 20.0) * 0.5 + 0.5;
            const highland = 186 + (nMid - 0.5) * 36 + (cRing - 0.5) * 26;
            const mareCol = 82 + (nMid - 0.5) * 18;
            const blend = Math.min(1, mariaMask * 1.5);
            const finalVal = highland * (1 - blend) + mareCol * blend;
            r = finalVal;
            g = finalVal * 0.99;
            b = finalVal * 0.97;
            bump = Math.max(0, Math.min(255, Math.floor(128 + (cRing - 0.5) * 85 * (1 - blend * 0.6))));
          } else if (type === 'mars') {
            // Rust iron-oxide highlands + dark basaltic Syrtis Major + Valles Marineris + polar caps
            const darkRegion = Math.max(0, (0.48 - nLow) * 2.2);
            const canyon = Math.exp(-Math.pow((lat + 12) / 4.5, 2)) * Math.exp(-Math.pow((lon + 65) / 36, 2));
            r = 198 - darkRegion * 68 - canyon * 48 + (nMid - 0.5) * 32;
            g = 96 - darkRegion * 34 - canyon * 26 + (nMid - 0.5) * 22;
            b = 58 - darkRegion * 18 - canyon * 15 + (nMid - 0.5) * 16;
            bump = Math.max(0, Math.min(255, Math.floor(132 + (nMid - 0.5) * 85 - canyon * 95)));

            if (absLat > 78 + (nMid - 0.5) * 8) {
              r = 242;
              g = 244;
              b = 248;
              bump = 152;
              rough = 170;
            }
          } else if (type === 'jupiter') {
            const turb = fbm3D(nx * 5.0, ny * 14.0, nz * 5.0, 4);
            const bandCoord = ny * 13.5 + (turb - 0.5) * 1.45;
            const band = Math.sin(bandCoord * Math.PI);
            // Great Red Spot at lat -22, lon -45
            const dLat = (lat + 22) / 8.5;
            const dLon = (lon + 45) / 16.0;
            const grsDist = Math.sqrt(dLat * dLat + dLon * dLon);
            const grsMask = Math.max(0, 1 - grsDist);

            r = 204 + band * 32 + (turb - 0.5) * 24;
            g = 174 + band * 34 + (turb - 0.5) * 22;
            b = 136 + band * 30 + (turb - 0.5) * 20;

            if (grsMask > 0) {
              const swirl = Math.sin(grsDist * 14.0 + turb * 4.0) * 0.5 + 0.5;
              r = r * (1 - grsMask) + (214 + swirl * 26) * grsMask;
              g = g * (1 - grsMask) + (96 + swirl * 34) * grsMask;
              b = b * (1 - grsMask) + (68 + swirl * 24) * grsMask;
            }
            rough = 185;
          } else if (type === 'io') {
            const caldera = Math.pow(nMid, 2.2);
            r = 232 + (nLow - 0.5) * 35 - caldera * 60;
            g = 198 + (nLow - 0.5) * 55 - caldera * 115;
            b = 58 + (nLow - 0.5) * 40 - caldera * 35;
            bump = Math.floor(128 + (nMid - 0.5) * 55);
          } else if (type === 'europa') {
            const ridge1 = Math.abs(Math.sin((nx * 6 + ny * 4 + nLow * 1.4) * Math.PI));
            const ridge2 = Math.abs(Math.cos((nz * 7 - ny * 5 + nMid * 1.2) * Math.PI));
            const lineae = Math.pow(Math.max(0, 1 - Math.min(ridge1, ridge2) * 5.5), 1.6);
            r = 228 - lineae * 62 + (nHigh - 0.5) * 14;
            g = 222 - lineae * 118 + (nHigh - 0.5) * 14;
            b = 208 - lineae * 134 + (nHigh - 0.5) * 14;
            bump = Math.floor(128 + lineae * 65);
            rough = 135;
          } else if (type === 'ganymede') {
            const darkTerrain = nLow > 0.48 ? 1 : 0;
            const grooves = Math.sin((nx * 14 + ny * 10 + nMid * 2) * Math.PI) * 0.5 + 0.5;
            const base = darkTerrain ? 108 + (nMid - 0.5) * 26 : 168 + grooves * 34;
            r = base * 1.02;
            g = base * 0.99;
            b = base * 0.94;
            bump = Math.floor(128 + (grooves - 0.5) * 65);
          } else if (type === 'callisto') {
            const sparkle = nHigh > 0.76 ? 78 : 0;
            const base = 94 + (nLow - 0.5) * 28 + sparkle;
            r = base * 1.04;
            g = base * 0.98;
            b = base * 0.9;
            bump = Math.floor(128 + (nMid - 0.5) * 80);
          } else if (type === 'saturn') {
            const band = Math.sin(ny * 18.0 + (nLow - 0.5) * 0.6);
            r = 224 + band * 18 + (nMid - 0.5) * 12;
            g = 198 + band * 18 + (nMid - 0.5) * 12;
            b = 148 + band * 16 + (nMid - 0.5) * 10;
            rough = 185;
          } else if (type === 'mimas') {
            // Herschel Crater on leading face
            const hDist = Math.sqrt(Math.pow(nx - 0.85, 2) + Math.pow(ny - 0.1, 2) + Math.pow(nz - 0.2, 2));
            const herschel = hDist < 0.45 ? Math.cos((hDist / 0.45) * Math.PI) : 0;
            const cField = Math.sin(nMid * 18.0) * 0.5 + 0.5;
            const base = 186 + (cField - 0.5) * 32 - herschel * 26;
            r = base;
            g = base * 1.01;
            b = base * 1.03;
            bump = Math.max(0, Math.min(255, Math.floor(128 + (cField - 0.5) * 70 - herschel * 95)));
            rough = 145;
          } else if (type === 'enceladus') {
            // Snow-white ice + blue-cyan south polar Tiger Stripes
            const southStripe = lat < -52 ? Math.pow(Math.abs(Math.sin((nx * 12 + nz * 12) * Math.PI)), 8) : 0;
            r = 244 - southStripe * 95 + (nHigh - 0.5) * 10;
            g = 247 - southStripe * 28 + (nHigh - 0.5) * 10;
            b = 252 - southStripe * 8 + (nHigh - 0.5) * 8;
            bump = Math.floor(128 + southStripe * 70 + (nMid - 0.5) * 30);
            rough = 110;
          } else if (type === 'tethys' || type === 'icy-grey') {
            const chasm = type === 'tethys' ? Math.exp(-Math.pow((lon - 20) / 7, 2)) * 0.8 : 0;
            const cField = Math.sin(nMid * 16.0) * 0.5 + 0.5;
            const base = 196 + (cField - 0.5) * 32 - chasm * 42;
            r = base;
            g = base * 1.01;
            b = base * 1.03;
            bump = Math.floor(128 + (cField - 0.5) * 65 - chasm * 80);
            rough = 135;
          } else if (type === 'titan') {
            const band = Math.sin(ny * 6.0 + (nLow - 0.5) * 0.8);
            r = 224 + band * 16 + (nMid - 0.5) * 14;
            g = 154 + band * 18 + (nMid - 0.5) * 14;
            b = 64 + band * 14 + (nMid - 0.5) * 12;
            rough = 190;
          } else if (type === 'iapetus') {
            // Two-tone yin-yang: dark Cassini Regio on leading hemisphere (nx > 0) vs bright ice
            const dichotomy = Math.max(0, Math.min(1, (nx + (nLow - 0.5) * 0.45) * 1.6 + 0.5));
            const ridge = Math.exp(-Math.pow(lat / 2.6, 2)) * 0.85;
            r = 226 * (1 - dichotomy) + 54 * dichotomy;
            g = 228 * (1 - dichotomy) + 44 * dichotomy;
            b = 232 * (1 - dichotomy) + 38 * dichotomy;
            bump = Math.min(255, Math.floor(128 + ridge * 115 + (nMid - 0.5) * 45));
          } else if (type === 'uranus') {
            const band = Math.sin(ny * 8.0);
            r = 146 + band * 12 + (nLow - 0.5) * 10;
            g = 216 + band * 12 + (nLow - 0.5) * 10;
            b = 232 + band * 10 + (nLow - 0.5) * 8;
            rough = 175;
          } else if (type === 'umbriel') {
            const wunda = Math.exp(-Math.pow((lat + 6) / 6, 2)) * Math.exp(-Math.pow((lon - 40) / 8, 2));
            const base = 86 + (nMid - 0.5) * 24 + wunda * 135;
            r = base;
            g = base * 1.01;
            b = base * 1.04;
            bump = Math.floor(128 + (nMid - 0.5) * 65);
          } else if (type === 'neptune') {
            const band = Math.sin(ny * 11.0 + (nMid - 0.5) * 0.9);
            const gds = Math.exp(-Math.pow((lat + 20) / 7, 2)) * Math.exp(-Math.pow((lon - 30) / 14, 2));
            const cirrus = nHigh > 0.8 ? 85 : 0;
            r = Math.min(255, 42 + band * 14 - gds * 22 + cirrus);
            g = Math.min(255, 92 + band * 18 - gds * 32 + cirrus);
            b = Math.min(255, 212 + band * 22 - gds * 42 + cirrus * 0.5);
            rough = 170;
          } else if (type === 'triton') {
            // Pinkish south polar nitrogen cap + greenish-blue cantaloupe terrain
            const southCap = lat < -18 + (nLow - 0.5) * 22;
            if (southCap) {
              r = 224 + (nMid - 0.5) * 18;
              g = 192 + (nMid - 0.5) * 18;
              b = 196 + (nMid - 0.5) * 18;
            } else {
              const dimple = Math.sin(nMid * 22.0) * 0.5 + 0.5;
              r = 164 + dimple * 28;
              g = 182 + dimple * 26;
              b = 188 + dimple * 24;
              bump = Math.floor(128 + (dimple - 0.5) * 65);
            }
            rough = 135;
          } else if (type === 'pluto') {
            // Bright Tombaugh Regio heart + dark reddish-brown Cthulhu Macula belt
            const heart = Math.exp(-Math.pow((lat + 8) / 24, 2)) * Math.exp(-Math.pow((lon - 15) / 28, 2));
            const cthulhu = Math.exp(-Math.pow((lat + 10) / 14, 2)) * Math.exp(-Math.pow((lon + 85) / 55, 2));
            r = 192 + heart * 52 - cthulhu * 98 + (nMid - 0.5) * 24;
            g = 164 + heart * 74 - cthulhu * 108 + (nMid - 0.5) * 22;
            b = 142 + heart * 92 - cthulhu * 96 + (nMid - 0.5) * 20;
            bump = Math.floor(128 + (nMid - 0.5) * 60 - heart * 35);
            rough = 145;
          } else if (type === 'charon') {
            // Grey icy terrain + dark reddish Mordor Macula north polar cap
            const mordor = lat > 52 + (nLow - 0.5) * 16 ? 1 : 0;
            const chasm = Math.exp(-Math.pow((lat + 4) / 4.5, 2)) * 0.75;
            const base = 152 + (nMid - 0.5) * 28 - chasm * 35;
            r = mordor ? 104 + (nMid - 0.5) * 20 : base;
            g = mordor ? 76 + (nMid - 0.5) * 16 : base * 0.99;
            b = mordor ? 68 + (nMid - 0.5) * 14 : base * 1.01;
            bump = Math.floor(128 + (nMid - 0.5) * 65 - chasm * 75);
          }

          const idx = (y * W + x) * 4;
          aImg.data[idx] = Math.max(0, Math.min(255, Math.floor(r)));
          aImg.data[idx + 1] = Math.max(0, Math.min(255, Math.floor(g)));
          aImg.data[idx + 2] = Math.max(0, Math.min(255, Math.floor(b)));
          aImg.data[idx + 3] = 255;

          bImg.data[idx] = bump;
          bImg.data[idx + 1] = bump;
          bImg.data[idx + 2] = bump;
          bImg.data[idx + 3] = 255;

          rImg.data[idx] = rough;
          rImg.data[idx + 1] = rough;
          rImg.data[idx + 2] = rough;
          rImg.data[idx + 3] = 255;
        }
      }

      aCtx.putImageData(aImg, 0, 0);
      bCtx.putImageData(bImg, 0, 0);
      rCtx.putImageData(rImg, 0, 0);

      const mapTex = new THREE.CanvasTexture(albedoCanvas);
      const bumpTex = new THREE.CanvasTexture(bumpCanvas);
      const roughTex = new THREE.CanvasTexture(roughCanvas);
      [mapTex, bumpTex, roughTex].forEach((t) => {
        t.wrapS = THREE.RepeatWrapping;
        t.wrapT = THREE.ClampToEdgeWrapping;
      });

      return { map: mapTex, bumpMap: bumpTex, roughnessMap: roughTex };
    }

    /**
     * Custom GLSL Rayleigh/Mie Fresnel Atmospheric Limb Scattering Halo
     */
    function createAtmosphereMesh(radius, config) {
      const geo = new THREE.SphereGeometry(radius * 1.036, 48, 32);
      const mat = new THREE.ShaderMaterial({
        transparent: true,
        blending: THREE.AdditiveBlending,
        side: THREE.BackSide,
        depthWrite: false,
        uniforms: {
          dayColor: { value: new THREE.Vector3(...config.dayColor) },
          sunsetColor: { value: new THREE.Vector3(...config.sunsetColor) },
          density: { value: config.density },
          power: { value: config.power }
        },
        vertexShader: `
          varying vec3 vNormalWorld;
          varying vec3 vWorldPos;
          void main() {
            vec4 worldPos = modelMatrix * vec4(position, 1.0);
            vWorldPos = worldPos.xyz;
            vNormalWorld = normalize(mat3(modelMatrix) * normal);
            gl_Position = projectionMatrix * viewMatrix * worldPos;
          }
        `,
        fragmentShader: `
          uniform vec3 dayColor;
          uniform vec3 sunsetColor;
          uniform float density;
          uniform float power;
          varying vec3 vNormalWorld;
          varying vec3 vWorldPos;
          void main() {
            vec3 viewDir = normalize(cameraPosition - vWorldPos);
            vec3 sunDir = normalize(-vWorldPos);
            float rim = pow(clamp(1.0 + dot(viewDir, vNormalWorld), 0.0, 1.0), power);
            float sunFactor = dot(vNormalWorld, sunDir);
            float dayWeight = smoothstep(-0.35, 0.55, sunFactor);
            float twilight = smoothstep(-0.45, 0.0, sunFactor) * (1.0 - smoothstep(0.0, 0.45, sunFactor));
            vec3 col = mix(dayColor, sunsetColor, twilight * 0.75) * (0.18 + 0.82 * dayWeight);
            gl_FragColor = vec4(col, rim * density * (0.25 + 0.75 * dayWeight));
          }
        `
      });
      return new THREE.Mesh(geo, mat);
    }

    /**
     * Generates realistic Saturn / Uranus rings with Cassini & Encke Divisions
     */
    function createPlanetaryRings(innerR, outerR, tintType = 'saturn') {
      const w = 512;
      const c = document.createElement('canvas');
      c.width = w;
      c.height = 16;
      const ctx = c.getContext('2d');
      const img = ctx.createImageData(w, 16);

      for (let x = 0; x < w; x++) {
        const u = x / (w - 1);
        let alpha = 0;
        let r = 225;
        let g = 205;
        let b = 168;

        if (tintType === 'uranus') {
          const narrow = Math.pow(Math.abs(Math.sin(u * 26.0)), 14);
          alpha = narrow * 0.45;
          r = 165;
          g = 195;
          b = 215;
        } else {
          // Saturn C, B, Cassini Division, A, Encke Gap, and F rings
          if (u < 0.16) {
            alpha = 0.22 + u * 1.1;
            r = 175; g = 160; b = 135;
          } else if (u < 0.56) {
            const fine = Math.sin(u * 190) * 0.12 + Math.cos(u * 75) * 0.08;
            alpha = 0.86 + fine;
            r = 236; g = 215; b = 175;
          } else if (u < 0.63) {
            // Cassini Division
            alpha = 0.05;
          } else if (u < 0.91) {
            const encke = Math.abs(u - 0.84) < 0.012 ? 0.08 : 0.68;
            alpha = encke + Math.sin(u * 150) * 0.08;
            r = 215; g = 196; b = 162;
          } else if (u > 0.96 && u < 0.985) {
            // Narrow F Ring
            alpha = 0.55;
          }
        }

        for (let y = 0; y < 16; y++) {
          const idx = (y * w + x) * 4;
          img.data[idx] = r;
          img.data[idx + 1] = g;
          img.data[idx + 2] = b;
          img.data[idx + 3] = Math.max(0, Math.min(255, Math.floor(alpha * 255)));
        }
      }
      ctx.putImageData(img, 0, 0);
      const tex = new THREE.CanvasTexture(c);

      const geo = new THREE.RingGeometry(innerR, outerR, 96, 1);
      const pos = geo.attributes.position;
      const uv = geo.attributes.uv;
      for (let i = 0; i < pos.count; i++) {
        const vx = pos.getX(i);
        const vy = pos.getY(i);
        const rad = Math.sqrt(vx * vx + vy * vy);
        uv.setXY(i, (rad - innerR) / (outerR - innerR), 0.5);
      }

      const mat = new THREE.MeshStandardMaterial({
        map: tex,
        transparent: true,
        side: THREE.DoubleSide,
        roughness: 0.85,
        metalness: 0.05
      });
      const mesh = new THREE.Mesh(geo, mat);
      mesh.rotation.x = Math.PI / 2;
      return mesh;
    }

    // =========================================================================
    // 5. BUILD 3D CELESTIAL BODIES, ORBIT PATHS & KEPLERIAN SOLVER
    // =========================================================================
    const celestialEntities = [];
    const entityById = {};
    const clickableHitMeshes = [];
    const orbitLines = [];

    function solveKeplerPosition(body, angle, outVec) {
      if (body.distance === 0) {
        outVec.set(0, 0, 0);
        return outVec;
      }

      const e = body.eccentricity || 0;
      const a = body.distance;
      // Solve Kepler's equation M = E - e sin(E)
      let E = angle;
      for (let iter = 0; iter < 4; iter++) {
        E = angle + e * Math.sin(E);
      }

      const xv = a * (Math.cos(E) - e);
      const zv = a * Math.sqrt(1 - e * e) * Math.sin(E);

      // Apply argument of periapsis, inclination, and ascending node
      const cosW = Math.cos(body.argPeriapsis || 0);
      const sinW = Math.sin(body.argPeriapsis || 0);
      const x1 = xv * cosW - zv * sinW;
      const z1 = xv * sinW + zv * cosW;

      const inc = body.inclination || 0;
      const y2 = z1 * Math.sin(inc);
      const z2 = z1 * Math.cos(inc);

      const omega = body.ascendingNode || 0;
      const cosO = Math.cos(omega);
      const sinO = Math.sin(omega);

      outVec.set(
        x1 * cosO - z2 * sinO,
        y2,
        x1 * sinO + z2 * cosO
      );
      return outVec;
    }

    SOLAR_BODIES.forEach((body, idx) => {
      const rootGroup = new THREE.Group();
      scene.add(rootGroup);

      const spinGroup = new THREE.Group();
      spinGroup.rotation.z = body.tilt || 0;
      rootGroup.add(spinGroup);

      const maps = generateBodyMaps(body.textureType);
      const geo = body.irregularSeed
        ? createIrregularGeometry(body.radius, body.irregularSeed)
        : new THREE.SphereGeometry(body.radius, body.category === 'planet' ? 64 : 36, body.category === 'planet' ? 48 : 24);

      if (body.oblateness && body.oblateness > 0) {
        geo.scale(1, 1 - body.oblateness, 1);
      }

      let mesh;
      if (body.id === 'sun') {
        const sunMat = new THREE.MeshBasicMaterial({ map: maps.map });
        mesh = new THREE.Mesh(geo, sunMat);
        const spriteMat = new THREE.SpriteMaterial({
          map: createSunGlowTexture(),
          color: 0xffaa44,
          transparent: true,
          blending: THREE.AdditiveBlending,
          depthWrite: false
        });
        const sprite = new THREE.Sprite(spriteMat);
        sprite.scale.set(body.radius * 4.6, body.radius * 4.6, 1);
        rootGroup.add(sprite);
      } else {
        const mat = new THREE.MeshStandardMaterial({
          map: maps.map,
          bumpMap: maps.bumpMap,
          bumpScale: body.bumpScale || 0.02,
          roughnessMap: maps.roughnessMap,
          roughness: 0.92,
          metalness: 0.04
        });
        mesh = new THREE.Mesh(geo, mat);
      }

      spinGroup.add(mesh);

      // Earth Dynamic Clouds & Night-Side City Lights
      let cloudMesh = null;
      if (body.hasClouds && maps.cloudMap) {
        const cGeo = new THREE.SphereGeometry(body.radius * 1.015, 64, 48);
        if (body.oblateness) cGeo.scale(1, 1 - body.oblateness, 1);
        const cMat = new THREE.MeshStandardMaterial({
          map: maps.cloudMap,
          transparent: true,
          opacity: 0.88,
          depthWrite: false,
          roughness: 0.9
        });
        cloudMesh = new THREE.Mesh(cGeo, cMat);
        spinGroup.add(cloudMesh);
      }

      if (body.hasCityLights && maps.nightMap) {
        const nGeo = new THREE.SphereGeometry(body.radius * 1.004, 64, 48);
        if (body.oblateness) nGeo.scale(1, 1 - body.oblateness, 1);
        const nMat = new THREE.ShaderMaterial({
          transparent: true,
          blending: THREE.AdditiveBlending,
          depthWrite: false,
          uniforms: {
            nightMap: { value: maps.nightMap }
          },
          vertexShader: `
            varying vec2 vUv;
            varying vec3 vNormalWorld;
            varying vec3 vWorldPos;
            void main() {
              vUv = uv;
              vec4 wp = modelMatrix * vec4(position, 1.0);
              vWorldPos = wp.xyz;
              vNormalWorld = normalize(mat3(modelMatrix) * normal);
              gl_Position = projectionMatrix * viewMatrix * wp;
            }
          `,
          fragmentShader: `
            uniform sampler2D nightMap;
            varying vec2 vUv;
            varying vec3 vNormalWorld;
            varying vec3 vWorldPos;
            void main() {
              vec3 sunDir = normalize(-vWorldPos);
              float nightMask = smoothstep(0.12, -0.25, dot(vNormalWorld, sunDir));
              vec4 lights = texture2D(nightMap, vUv);
              gl_FragColor = vec4(lights.rgb * nightMask * 1.35, 1.0);
            }
          `
        });
        const nightMesh = new THREE.Mesh(nGeo, nMat);
        spinGroup.add(nightMesh);
      }

      if (body.atmosphere) {
        const atmo = createAtmosphereMesh(body.radius, body.atmosphere);
        if (body.oblateness) atmo.scale.set(1, 1 - body.oblateness, 1);
        spinGroup.add(atmo);
      }

      if (body.hasRings) {
        const rings = createPlanetaryRings(body.ringInner, body.ringOuter, body.ringTint || 'saturn');
        spinGroup.add(rings);
      }

      // Enlarged invisible hitbox so small moons & planets are effortless to click/tap
      const hitRadius = Math.max(body.radius * 1.45, body.category === 'moon' ? 0.58 : 1.05);
      const hitGeo = new THREE.SphereGeometry(hitRadius, 16, 12);
      const hitMat = new THREE.MeshBasicMaterial({ visible: false });
      const hitMesh = new THREE.Mesh(hitGeo, hitMat);
      hitMesh.userData.bodyIndex = idx;
      rootGroup.add(hitMesh);
      clickableHitMeshes.push(hitMesh);

      // Orbit Path Line
      let orbitLine = null;
      if (body.distance > 0) {
        const pts = [];
        const segs = body.category === 'planet' ? 180 : 96;
        const tmp = new THREE.Vector3();
        for (let s = 0; s <= segs; s++) {
          solveKeplerPosition(body, (s / segs) * Math.PI * 2, tmp);
          pts.push(tmp.clone());
        }
        const oGeo = new THREE.BufferGeometry().setFromPoints(pts);
        const oMat = new THREE.LineBasicMaterial({
          color: body.category === 'planet' ? 0x444444 : 0x331111,
          transparent: true,
          opacity: body.category === 'planet' ? 0.65 : 0.5
        });
        orbitLine = new THREE.Line(oGeo, oMat);
        scene.add(orbitLine);
        orbitLines.push(orbitLine);
      }

      const entity = {
        index: idx,
        data: body,
        rootGroup,
        spinGroup,
        mesh,
        cloudMesh,
        orbitLine,
        angle: idx * 1.17,
        worldPos: new THREE.Vector3()
      };

      celestialEntities.push(entity);
      entityById[body.id] = entity;
    });

    // Main Asteroid Belt Particle Ring around Ceres's orbital zone
    (function createAsteroidBelt() {
      const count = 1400;
      const positions = new Float32Array(count * 3);
      for (let i = 0; i < count; i++) {
        const a = Math.random() * Math.PI * 2;
        const r = 42.2 + (Math.random() - 0.5) * 5.2;
        const y = (Math.random() - 0.5) * 1.6;
        positions[i * 3] = Math.cos(a) * r;
        positions[i * 3 + 1] = y;
        positions[i * 3 + 2] = Math.sin(a) * r;
      }
      const geo = new THREE.BufferGeometry();
      geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      const mat = new THREE.PointsMaterial({
        color: 0x888888,
        size: 0.24,
        transparent: true,
        opacity: 0.65
      });
      scene.add(new THREE.Points(geo, mat));
    })();

    // High-Tech 3D Crimson Targeting Reticle Ring
    const targetReticleGroup = new THREE.Group();
    const reticleRing = new THREE.Mesh(
      new THREE.RingGeometry(1.18, 1.28, 48),
      new THREE.MeshBasicMaterial({
        color: 0xff0000,
        side: THREE.DoubleSide,
        transparent: true,
        opacity: 0.92
      })
    );
    reticleRing.rotation.x = Math.PI / 2;
    targetReticleGroup.add(reticleRing);
    scene.add(targetReticleGroup);

    // =========================================================================
    // 6. RIGID CAMERA TARGET LOCK, ORBIT CONTROLS & RAYCASTING
    // =========================================================================
    const camSpherical = {
      radius: 128,
      theta: 0.52,
      phi: 0.96
    };

    const cameraFocusPoint = new THREE.Vector3(0, 0, 0);
    const desiredFocusPoint = new THREE.Vector3(0, 0, 0);

    const raycaster = new THREE.Raycaster();
    const pointerNDC = new THREE.Vector2();
    let isPointerDown = false;
    let pointerMovedDist = 0;
    let lastPointerX = 0;
    let lastPointerY = 0;
    let lastPinchDist = null;

    const hoverTooltipEl = document.getElementById('hoverTooltip');
    const hoverTooltipTextEl = document.getElementById('hoverTooltipText');

    function updatePointerNDC(clientX, clientY) {
      const rect = canvas.getBoundingClientRect();
      pointerNDC.x = ((clientX - rect.left) / rect.width) * 2 - 1;
      pointerNDC.y = -((clientY - rect.top) / rect.height) * 2 + 1;
    }

    function pickCelestialUnderPointer(clientX, clientY) {
      updatePointerNDC(clientX, clientY);
      raycaster.setFromCamera(pointerNDC, camera);
      const hits = raycaster.intersectObjects(clickableHitMeshes, false);
      if (hits.length > 0) {
        return hits[0].object.userData.bodyIndex;
      }
      return -1;
    }

    canvas.addEventListener('pointerdown', (e) => {
      isPointerDown = true;
      pointerMovedDist = 0;
      lastPointerX = e.clientX;
      lastPointerY = e.clientY;
    });

    window.addEventListener('pointermove', (e) => {
      if (isPointerDown) {
        const dx = e.clientX - lastPointerX;
        const dy = e.clientY - lastPointerY;
        pointerMovedDist += Math.hypot(dx, dy);
        lastPointerX = e.clientX;
        lastPointerY = e.clientY;

        camSpherical.theta -= dx * 0.0065;
        camSpherical.phi = Math.max(0.14, Math.min(Math.PI - 0.14, camSpherical.phi - dy * 0.0065));
        hoverTooltipEl.classList.add('hidden');
        return;
      }

      if (e.pointerType === 'mouse') {
        const hoveredIdx = pickCelestialUnderPointer(e.clientX, e.clientY);
        if (hoveredIdx >= 0) {
          canvas.style.cursor = 'pointer';
          hoverTooltipTextEl.textContent = SOLAR_BODIES[hoveredIdx].name;
          hoverTooltipEl.style.left = `${Math.min(window.innerWidth - 190, e.clientX + 14)}px`;
          hoverTooltipEl.style.top = `${Math.max(12, e.clientY - 18)}px`;
          hoverTooltipEl.classList.remove('hidden');
        } else {
          canvas.style.cursor = 'grab';
          hoverTooltipEl.classList.add('hidden');
        }
      }
    });

    window.addEventListener('pointerup', (e) => {
      if (isPointerDown && pointerMovedDist < 7) {
        const pickedIdx = pickCelestialUnderPointer(e.clientX, e.clientY);
        if (pickedIdx >= 0) {
          selectCelestialBody(pickedIdx, true);
        }
      }
      isPointerDown = false;
    });

    canvas.addEventListener(
      'wheel',
      (e) => {
        e.preventDefault();
        const factor = e.deltaY > 0 ? 1.12 : 0.89;
        const minR = selectedIndex >= 0 ? Math.max(0.75, SOLAR_BODIES[selectedIndex].radius * 2.3) : 6.0;
        camSpherical.radius = Math.max(minR, Math.min(440, camSpherical.radius * factor));
      },
      { passive: false }
    );

    // Touch pinch-to-zoom support
    canvas.addEventListener(
      'touchmove',
      (e) => {
        if (e.touches.length === 2) {
          e.preventDefault();
          const dx = e.touches[0].clientX - e.touches[1].clientX;
          const dy = e.touches[0].clientY - e.touches[1].clientY;
          const dist = Math.hypot(dx, dy);
          if (lastPinchDist !== null && dist > 0) {
            const scale = lastPinchDist / dist;
            const minR = selectedIndex >= 0 ? Math.max(0.75, SOLAR_BODIES[selectedIndex].radius * 2.3) : 6.0;
            camSpherical.radius = Math.max(minR, Math.min(440, camSpherical.radius * scale));
          }
          lastPinchDist = dist;
        }
      },
      { passive: false }
    );

    canvas.addEventListener('touchend', () => {
      lastPinchDist = null;
    });

    // =========================================================================
    // 7. HUD INSPECTOR, ROSTER NAVIGATION & ANIMATION LOOP
    // =========================================================================
    const inspectorPanelEl = document.getElementById('inspectorPanel');
    const reopenInspectorBtn = document.getElementById('reopenInspectorBtn');
    const collapseInspectorBtn = document.getElementById('collapseInspectorBtn');

    const inspectorSectorBadgeEl = document.getElementById('inspectorSectorBadge');
    const bodyNameEl = document.getElementById('bodyName');
    const viewPhotoBtn = document.getElementById('viewPhotoBtn');
    const viewSchematicBtn = document.getElementById('viewSchematicBtn');
    const photoViewportEl = document.getElementById('photoViewport');
    const schematicViewportEl = document.getElementById('schematicViewport');
    const imageLoadingOverlayEl = document.getElementById('imageLoadingOverlay');
    const imageCatalogBadgeEl = document.getElementById('imageCatalogBadge');
    const bodyImageEl = document.getElementById('bodyImage');
    const imageCaptionTextEl = document.getElementById('imageCaptionText');

    const jumpParentBtn = document.getElementById('jumpParentBtn');
    const bodyDescriptionEl = document.getElementById('bodyDescription');
    const planetMoonsBoxEl = document.getElementById('planetMoonsBox');
    const planetMoonsCountEl = document.getElementById('planetMoonsCount');
    const planetMoonsListEl = document.getElementById('planetMoonsList');

    const subFeaturesTabsEl = document.getElementById('subFeaturesTabs');
    const subFeatureTitleEl = document.getElementById('subFeatureTitle');
    const subFeatureDetailEl = document.getElementById('subFeatureDetail');

    const statDiameterEl = document.getElementById('statDiameter');
    const statDistanceEl = document.getElementById('statDistance');
    const statPeriodEl = document.getElementById('statPeriod');
    const statTempEl = document.getElementById('statTemp');
    const statEccentricityEl = document.getElementById('statEccentricity');
    const statObliquityEl = document.getElementById('statObliquity');

    const rosterBarEl = document.getElementById('rosterBar');
    const filterAllBtn = document.getElementById('filterAllBtn');
    const filterPlanetsBtn = document.getElementById('filterPlanetsBtn');
    const filterMoonsBtn = document.getElementById('filterMoonsBtn');

    function setMediaMode(mode) {
      if (mode === 'photo') {
        photoViewportEl.classList.remove('hidden');
        schematicViewportEl.classList.add('hidden');
        viewPhotoBtn.className = 'px-2.5 py-1 rounded-md bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors';
        viewSchematicBtn.className = 'px-2.5 py-1 rounded-md text-[#ffffff]/70 hover:text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors';
      } else {
        photoViewportEl.classList.add('hidden');
        schematicViewportEl.classList.remove('hidden');
        viewSchematicBtn.className = 'px-2.5 py-1 rounded-md bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors';
        viewPhotoBtn.className = 'px-2.5 py-1 rounded-md text-[#ffffff]/70 hover:text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer transition-colors';
      }
    }

    viewPhotoBtn.addEventListener('click', () => {
      setMediaMode('photo');
      playHudTone(720, 0.05);
    });

    viewSchematicBtn.addEventListener('click', () => {
      setMediaMode('schematic');
      playHudTone(580, 0.05);
    });

    function loadRealSpacecraftImage(body) {
      imageCatalogBadgeEl.textContent = body.nasaCatalog || 'NASA // JPL ARCHIVE';
      bodyImageEl.alt = `${body.name} — Real Spacecraft Photograph`;

      if (resolvedImageCache[body.id]) {
        imageLoadingOverlayEl.classList.add('hidden');
        bodyImageEl.style.opacity = '1';
        bodyImageEl.src = resolvedImageCache[body.id];
        return;
      }

      imageLoadingOverlayEl.classList.remove('hidden');
      bodyImageEl.style.opacity = '0.35';
      const targetId = body.id;

      resolveRealSpacecraftUrl(body).then((verifiedUrl) => {
        if (SOLAR_BODIES[selectedIndex].id === targetId) {
          bodyImageEl.onload = () => {
            imageLoadingOverlayEl.classList.add('hidden');
            bodyImageEl.style.opacity = '1';
          };
          bodyImageEl.onerror = async () => {
            bodyImageEl.onerror = null;
            const snapUrl = await createOfflineOrbitalCaptureUrl(body);
            resolvedImageCache[targetId] = snapUrl;
            if (SOLAR_BODIES[selectedIndex].id === targetId) {
              bodyImageEl.src = snapUrl;
              imageLoadingOverlayEl.classList.add('hidden');
              bodyImageEl.style.opacity = '1';
            }
          };
          bodyImageEl.src = verifiedUrl;
        }
      });
    }

    function renderSubFeatures(body, activeFeatureIdx = 0) {
      subFeaturesTabsEl.innerHTML = '';
      body.subFeatures.forEach((feat, fIdx) => {
        const isActive = fIdx === activeFeatureIdx;
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.className = isActive
          ? 'px-2.5 py-1 rounded bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer'
          : 'px-2.5 py-1 rounded border border-[#ffffff]/20 hover:border-[#ff0000] text-[#ffffff]/80 font-mono text-[10px] uppercase cursor-pointer transition-colors';
        btn.textContent = feat.title;

        btn.addEventListener('click', () => {
          renderSubFeatures(body, fIdx);
          playHudTone(680 + fIdx * 60, 0.045);
        });

        subFeaturesTabsEl.appendChild(btn);
      });

      const currentFeat = body.subFeatures[activeFeatureIdx] || body.subFeatures[0];
      subFeatureTitleEl.textContent = currentFeat.title;
      subFeatureDetailEl.textContent = currentFeat.detail;
    }

    function selectCelestialBody(index, focusCamera = true) {
      if (index < 0 || index >= SOLAR_BODIES.length) return;
      selectedIndex = index;
      const entity = celestialEntities[index];
      const body = entity.data;

      playHudTone(620, 0.06);

      // Highlight active orbit ring
      celestialEntities.forEach((ent, i) => {
        if (ent.orbitLine) {
          const isSel = i === index;
          ent.orbitLine.material.color.setHex(
            isSel ? 0xff0000 : ent.data.category === 'planet' ? 0x444444 : 0x331111
          );
          ent.orbitLine.material.opacity = isSel ? 0.95 : ent.data.category === 'planet' ? 0.65 : 0.48;
        }
      });

      // Update Inspector Telemetry
      inspectorSectorBadgeEl.textContent = body.sector;
      bodyNameEl.textContent = body.name;
      imageCaptionTextEl.textContent = `${body.nasaCatalog} // ${body.shortName}`;
      bodyDescriptionEl.textContent = body.description;

      statDiameterEl.textContent = body.diameter;
      statDistanceEl.textContent = body.distanceText;
      statPeriodEl.textContent = body.periodText;
      statTempEl.textContent = body.tempText;
      statEccentricityEl.textContent = body.eccentricityText;
      statObliquityEl.textContent = body.obliquityText;

      // Load Real Spacecraft Photo & Structural Schematic
      loadRealSpacecraftImage(body);
      schematicViewportEl.innerHTML = generateStructuralSVG(body);

      // Show Parent Planet Jump Button if this body is a Moon
      if (body.category === 'moon' && body.parentId && entityById[body.parentId]) {
        const parentEnt = entityById[body.parentId];
        jumpParentBtn.textContent = `↑ ${parentEnt.data.shortName}`;
        jumpParentBtn.classList.remove('hidden');
        jumpParentBtn.onclick = () => selectCelestialBody(parentEnt.index, true);
      } else {
        jumpParentBtn.classList.add('hidden');
      }

      // Show Orbiting Moons Matrix if this body is a Planet with Moons
      const childMoons = celestialEntities.filter(
        (ent) => ent.data.category === 'moon' && ent.data.parentId === body.id
      );

      if (childMoons.length > 0) {
        planetMoonsBoxEl.classList.remove('hidden');
        planetMoonsCountEl.textContent = `${childMoons.length} ${childMoons.length === 1 ? 'MOON' : 'MOONS'}`;
        planetMoonsListEl.innerHTML = '';
        childMoons.forEach((moonEnt) => {
          const mBtn = document.createElement('button');
          mBtn.type = 'button';
          mBtn.className = 'inline-flex items-center gap-1.5 px-2.5 py-1 rounded border border-[#ff0000]/60 bg-[#ff0000]/12 hover:bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-semibold uppercase cursor-pointer transition-colors';

          const thumbImg = document.createElement('img');
          thumbImg.alt = moonEnt.data.shortName;
          thumbImg.className = 'w-4 h-4 rounded-full object-cover border border-[#ffffff]/40 bg-[#000000] shrink-0';
          resolveRealSpacecraftUrl(moonEnt.data).then((url) => {
            thumbImg.src = url;
          });

          const labelSpan = document.createElement('span');
          labelSpan.textContent = moonEnt.data.shortName;

          mBtn.appendChild(thumbImg);
          mBtn.appendChild(labelSpan);
          mBtn.addEventListener('click', () => {
            selectCelestialBody(moonEnt.index, true);
          });
          planetMoonsListEl.appendChild(mBtn);
        });
      } else {
        planetMoonsBoxEl.classList.add('hidden');
      }

      renderSubFeatures(body, 0);
      renderRosterBar();

      // Zoom camera smoothly to appropriate framing distance for the selected planet/moon
      if (focusCamera) {
        const idealDist = body.id === 'sun'
          ? 24
          : Math.max(1.35, body.radius * (body.hasRings ? 5.2 : 4.2));
        camSpherical.radius = idealDist;
      }

      // Re-open inspector if minimized
      inspectorPanelEl.classList.remove('hidden');
      reopenInspectorBtn.classList.add('hidden');
    }

    function renderRosterBar() {
      rosterBarEl.innerHTML = '';
      celestialEntities.forEach((ent) => {
        const body = ent.data;
        if (activeRosterFilter === 'planets' && body.category !== 'planet') return;
        if (activeRosterFilter === 'moons' && body.category !== 'moon') return;

        const isSelected = ent.index === selectedIndex;
        const btn = document.createElement('button');
        btn.type = 'button';
        btn.className = isSelected
          ? 'px-2.5 py-1 rounded bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase tracking-wider shrink-0 shadow-[0_0_12px_rgba(255,0,0,0.6)] cursor-pointer'
          : 'px-2.5 py-1 rounded border border-[#ffffff]/15 hover:border-[#ff0000] text-[#ffffff]/80 hover:text-[#ffffff] font-mono text-[10px] uppercase tracking-wider shrink-0 transition-colors cursor-pointer';
        btn.textContent = body.shortName;

        btn.addEventListener('click', () => {
          selectCelestialBody(ent.index, true);
        });

        rosterBarEl.appendChild(btn);
        if (isSelected) {
          btn.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
        }
      });
    }

    // Filter Buttons
    function setRosterFilter(filter) {
      activeRosterFilter = filter;
      const activeCls = 'px-2 py-1 rounded bg-[#ff0000] text-[#ffffff] font-mono text-[10px] font-bold uppercase cursor-pointer';
      const idleCls = 'px-2 py-1 rounded border border-[#ffffff]/20 hover:border-[#ff0000] text-[#ffffff]/80 font-mono text-[10px] uppercase cursor-pointer';

      filterAllBtn.className = filter === 'all' ? activeCls : idleCls;
      filterPlanetsBtn.className = filter === 'planets' ? activeCls : idleCls;
      filterMoonsBtn.className = filter === 'moons' ? activeCls : idleCls;

      renderRosterBar();
      playHudTone(560, 0.045);
    }

    filterAllBtn.addEventListener('click', () => setRosterFilter('all'));
    filterPlanetsBtn.addEventListener('click', () => setRosterFilter('planets'));
    filterMoonsBtn.addEventListener('click', () => setRosterFilter('moons'));

    // Top & Bottom HUD Controls
    const speedSlider = document.getElementById('speedSlider');
    const speedValueLabel = document.getElementById('speedValueLabel');
    speedSlider.addEventListener('input', (e) => {
      simulationSpeed = parseFloat(e.target.value);
      speedValueLabel.textContent = `${simulationSpeed.toFixed(1)}x`;
    });

    const toggleOrbitsBtn = document.getElementById('toggleOrbitsBtn');
    toggleOrbitsBtn.addEventListener('click', () => {
      showOrbits = !showOrbits;
      orbitLines.forEach((l) => {
        l.visible = showOrbits;
      });
      toggleOrbitsBtn.textContent = `ORBITS: ${showOrbits ? 'ON' : 'OFF'}`;
      toggleOrbitsBtn.className = showOrbits
        ? 'px-2.5 py-1.5 rounded-lg border border-[#ff0000] bg-[#ff0000]/15 text-[#ffffff] font-mono text-[11px] uppercase tracking-wider hover:bg-[#ff0000] transition-colors cursor-pointer'
        : 'px-2.5 py-1.5 rounded-lg border border-[#ffffff]/25 text-[#ffffff]/70 font-mono text-[11px] uppercase tracking-wider hover:border-[#ff0000] transition-colors cursor-pointer';
      playHudTone(600, 0.05);
    });

    const toggleAudioBtn = document.getElementById('toggleAudioBtn');
    toggleAudioBtn.addEventListener('click', () => {
      audioEnabled = !audioEnabled;
      toggleAudioBtn.textContent = `SFX: ${audioEnabled ? 'ON' : 'OFF'}`;
      if (audioEnabled) playHudTone(760, 0.06);
    });

    document.getElementById('resetViewBtn').addEventListener('click', () => {
      selectCelestialBody(0, false);
      camSpherical.radius = 148;
      camSpherical.theta = 0.52;
      camSpherical.phi = 0.94;
      playHudTone(500, 0.08);
    });

    document.getElementById('prevBodyBtn').addEventListener('click', () => {
      const nextIdx = (selectedIndex - 1 + SOLAR_BODIES.length) % SOLAR_BODIES.length;
      selectCelestialBody(nextIdx, true);
    });

    document.getElementById('nextBodyBtn').addEventListener('click', () => {
      const nextIdx = (selectedIndex + 1) % SOLAR_BODIES.length;
      selectCelestialBody(nextIdx, true);
    });

    collapseInspectorBtn.addEventListener('click', () => {
      inspectorPanelEl.classList.add('hidden');
      reopenInspectorBtn.classList.remove('hidden');
      playHudTone(460, 0.05);
    });

    reopenInspectorBtn.addEventListener('click', () => {
      inspectorPanelEl.classList.remove('hidden');
      reopenInspectorBtn.classList.add('hidden');
      playHudTone(680, 0.05);
    });

    document.getElementById('zoomInBtn').addEventListener('click', () => {
      const minR = selectedIndex >= 0 ? Math.max(0.75, SOLAR_BODIES[selectedIndex].radius * 2.3) : 6.0;
      camSpherical.radius = Math.max(minR, camSpherical.radius * 0.8);
      playHudTone(720, 0.04);
    });

    document.getElementById('zoomOutBtn').addEventListener('click', () => {
      camSpherical.radius = Math.min(440, camSpherical.radius * 1.25);
      playHudTone(540, 0.04);
    });

    window.addEventListener('resize', () => {
      const w = canvasWrapper.clientWidth;
      const h = canvasWrapper.clientHeight;
      camera.aspect = w / h;
      camera.updateProjectionMatrix();
      renderer.setSize(w, h);
    });

    // =========================================================================
    // 8. MAIN REAL-TIME SIMULATION & RIGID CAMERA TRACKING LOOP
    // =========================================================================
    const tmpLocalPos = new THREE.Vector3();

    function animate() {
      requestAnimationFrame(animate);

      // 1. Update Planets first
      for (let i = 0; i < celestialEntities.length; i++) {
        const ent = celestialEntities[i];
        const body = ent.data;
        if (body.category !== 'planet') continue;

        ent.angle += body.orbitSpeed * simulationSpeed * 0.35;
        solveKeplerPosition(body, ent.angle, ent.worldPos);
        ent.rootGroup.position.copy(ent.worldPos);

        if (body.rotationSpeed) {
          ent.spinGroup.rotation.y += body.rotationSpeed * Math.max(0.25, simulationSpeed);
        }
        if (ent.cloudMesh) {
          ent.cloudMesh.rotation.y += body.rotationSpeed * 0.38 * Math.max(0.25, simulationSpeed);
        }
      }

      // 2. Update Moons relative to their Parent Planets
      for (let i = 0; i < celestialEntities.length; i++) {
        const ent = celestialEntities[i];
        const body = ent.data;
        if (body.category !== 'moon') continue;

        const parentEnt = entityById[body.parentId];
        if (!parentEnt) continue;

        ent.angle += body.orbitSpeed * simulationSpeed * 0.35;
        solveKeplerPosition(body, ent.angle, tmpLocalPos);
        ent.worldPos.copy(parentEnt.worldPos).add(tmpLocalPos);
        ent.rootGroup.position.copy(ent.worldPos);

        if (ent.orbitLine) {
          ent.orbitLine.position.copy(parentEnt.worldPos);
        }

        // Tidally locked moons always keep the exact same face pointed toward their parent planet
        if (body.tidallyLocked) {
          ent.spinGroup.rotation.y = Math.atan2(
            parentEnt.worldPos.x - ent.worldPos.x,
            parentEnt.worldPos.z - ent.worldPos.z
          );
        } else if (body.rotationSpeed) {
          ent.spinGroup.rotation.y += body.rotationSpeed * Math.max(0.25, simulationSpeed);
        }
      }

      // 3. Rigid 1:1 Camera Target Lock on Selected Planet or Moon
      const targetEntity = celestialEntities[selectedIndex];
      if (targetEntity) {
        desiredFocusPoint.copy(targetEntity.worldPos);
        cameraFocusPoint.copy(desiredFocusPoint);

        targetReticleGroup.position.copy(targetEntity.worldPos);
        const rScale = Math.max(0.35, targetEntity.data.radius * (targetEntity.data.hasRings ? 2.45 : 1.35));
        targetReticleGroup.scale.set(rScale, rScale, rScale);
        targetReticleGroup.rotation.y += 0.015;
      }

      // Compute Spherical Camera Offset around locked target
      const sinPhi = Math.sin(camSpherical.phi);
      camera.position.set(
        cameraFocusPoint.x + camSpherical.radius * sinPhi * Math.sin(camSpherical.theta),
        cameraFocusPoint.y + camSpherical.radius * Math.cos(camSpherical.phi),
        cameraFocusPoint.z + camSpherical.radius * sinPhi * Math.cos(camSpherical.theta)
      );
      camera.lookAt(cameraFocusPoint);

      starField.rotation.y += 0.00004;
      renderer.render(scene, camera);
    }

    window.onload = function () {
      renderRosterBar();
      selectCelestialBody(0, false);
      prewarmAllSpacecraftImages();
      camSpherical.radius = 128;
      camSpherical.theta = 0.52;
      camSpherical.phi = 0.96;
      animate();
    };
  </script>
</body>
</html># space
