# Moon Phase Wallpaper App - Testing Specification

## 1. Test Strategy Overview

### 1.1 Testing Pyramid

```
         ┌─────┐
         │ E2E │      5% - End-to-End Tests
        ┌┴─────┴┐
        │  INT  │     15% - Integration Tests  
      ┌─┴───────┴─┐
      │   UNIT    │   80% - Unit Tests
    └─────────────┘
```

### 1.2 Test Coverage Goals

- **Unit Tests**: 90% code coverage
- **Integration Tests**: All critical paths covered
- **E2E Tests**: Happy paths and key error scenarios
- **Performance Tests**: Response time < 2s, memory < 100MB
- **Platform Tests**: Android 7.0+ and iOS 14.0+

## 2. Unit Test Specifications

### 2.1 Moon Calculator Tests

```typescript
describe('MoonCalculator', () => {
    let calculator: MoonCalculator;
    
    beforeEach(() => {
        calculator = new MoonCalculator();
    });
    
    describe('calculatePhase', () => {
        it('should calculate new moon correctly', () => {
            // Known new moon: January 13, 2021, 05:00 UTC
            const date = new Date('2021-01-13T05:00:00Z');
            const location = { latitude: 51.5074, longitude: -0.1278 }; // London
            
            const result = calculator.calculatePhase(date, location);
            
            expect(result.phase).toBe(MoonPhase.NEW_MOON);
            expect(result.illumination).toBeLessThan(1);
        });
        
        it('should calculate full moon correctly', () => {
            // Known full moon: January 28, 2021, 19:16 UTC
            const date = new Date('2021-01-28T19:16:00Z');
            const location = { latitude: 51.5074, longitude: -0.1278 };
            
            const result = calculator.calculatePhase(date, location);
            
            expect(result.phase).toBe(MoonPhase.FULL_MOON);
            expect(result.illumination).toBeGreaterThan(99);
        });
        
        it('should handle timezone differences', () => {
            const date = new Date('2021-01-28T23:00:00Z');
            const tokyo = { latitude: 35.6762, longitude: 139.6503 };
            const newYork = { latitude: 40.7128, longitude: -74.0060 };
            
            const tokyoResult = calculator.calculatePhase(date, tokyo);
            const nyResult = calculator.calculatePhase(date, newYork);
            
            // Different local dates might show different phases
            expect(tokyoResult.ageDays).not.toBe(nyResult.ageDays);
        });
        
        it('should calculate all 8 moon phases', () => {
            const phases = new Set<MoonPhase>();
            const startDate = new Date('2021-01-01');
            const location = { latitude: 0, longitude: 0 }; // Equator
            
            for (let i = 0; i < 30; i++) {
                const date = new Date(startDate);
                date.setDate(date.getDate() + i);
                const result = calculator.calculatePhase(date, location);
                phases.add(result.phase);
            }
            
            expect(phases.size).toBeGreaterThanOrEqual(4);
        });
    });
    
    describe('calculateIllumination', () => {
        it('should return 0% for new moon position', () => {
            const illumination = calculator.calculateIllumination(0);
            expect(illumination).toBe(0);
        });
        
        it('should return 100% for full moon position', () => {
            const illumination = calculator.calculateIllumination(0.5);
            expect(illumination).toBe(100);
        });
        
        it('should return ~50% for quarter positions', () => {
            const firstQuarter = calculator.calculateIllumination(0.25);
            const lastQuarter = calculator.calculateIllumination(0.75);
            
            expect(firstQuarter).toBeCloseTo(50, 0);
            expect(lastQuarter).toBeCloseTo(50, 0);
        });
    });
});
```

### 2.2 Wallpaper Renderer Tests

```typescript
describe('WallpaperRenderer', () => {
    let renderer: WallpaperRenderer;
    let mockThemeManager: jest.Mocked<ThemeManager>;
    
    beforeEach(() => {
        mockThemeManager = createMockThemeManager();
        renderer = new WallpaperRenderer(mockThemeManager);
    });
    
    describe('render', () => {
        it('should render wallpaper with correct dimensions', async () => {
            const moonData = createMockMoonData(MoonPhase.FULL_MOON);
            const theme = createMockTheme('realistic');
            const dimensions = { width: 1080, height: 1920 };
            
            const result = await renderer.render(moonData, theme, dimensions);
            
            expect(result.width).toBe(1080);
            expect(result.height).toBe(1920);
            expect(result.format).toBe('ARGB_8888');
        });
        
        it('should apply theme-specific rendering', async () => {
            const moonData = createMockMoonData(MoonPhase.CRESCENT);
            const dimensions = { width: 1080, height: 1920 };
            
            const realisticSpy = jest.spyOn(renderer, 'renderRealistic');
            const artisticSpy = jest.spyOn(renderer, 'renderArtistic');
            const minimalSpy = jest.spyOn(renderer, 'renderMinimal');
            
            // Test each theme
            await renderer.render(moonData, createMockTheme('realistic'), dimensions);
            expect(realisticSpy).toHaveBeenCalled();
            
            await renderer.render(moonData, createMockTheme('artistic'), dimensions);
            expect(artisticSpy).toHaveBeenCalled();
            
            await renderer.render(moonData, createMockTheme('minimal'), dimensions);
            expect(minimalSpy).toHaveBeenCalled();
        });
        
        it('should handle rendering errors gracefully', async () => {
            const moonData = createMockMoonData(MoonPhase.FULL_MOON);
            const theme = createMockTheme('realistic');
            const dimensions = { width: -1, height: -1 }; // Invalid
            
            await expect(renderer.render(moonData, theme, dimensions))
                .rejects.toThrow(RenderingError);
        });
    });
    
    describe('renderRealistic', () => {
        it('should apply phase shadow correctly', async () => {
            const fullMoon = createMockMoonData(MoonPhase.FULL_MOON);
            const newMoon = createMockMoonData(MoonPhase.NEW_MOON);
            const size = 500;
            
            const fullMoonImage = await renderer.renderRealistic(fullMoon, size);
            const newMoonImage = await renderer.renderRealistic(newMoon, size);
            
            // Full moon should have minimal shadow
            const fullMoonBrightness = calculateAverageBrightness(fullMoonImage);
            const newMoonBrightness = calculateAverageBrightness(newMoonImage);
            
            expect(fullMoonBrightness).toBeGreaterThan(newMoonBrightness);
        });
    });
});
```

### 2.3 Location Service Tests

```typescript
describe('LocationService', () => {
    let service: LocationService;
    let mockDatabase: jest.Mocked<CapitalCityDatabase>;
    
    beforeEach(() => {
        mockDatabase = createMockDatabase();
        service = new LocationService(mockDatabase);
    });
    
    describe('searchCapitals', () => {
        it('should find exact matches', async () => {
            mockDatabase.search.mockResolvedValue([
                { name: 'London', country: 'United Kingdom', latitude: 51.5074, longitude: -0.1278 }
            ]);
            
            const results = await service.searchCapitals('London');
            
            expect(results).toHaveLength(1);
            expect(results[0].name).toBe('London');
        });
        
        it('should find fuzzy matches', async () => {
            mockDatabase.search.mockResolvedValue([
                { name: 'Tokyo', country: 'Japan', latitude: 35.6762, longitude: 139.6503 }
            ]);
            
            const results = await service.searchCapitals('Tokio'); // Misspelled
            
            expect(results).toHaveLength(1);
            expect(results[0].name).toBe('Tokyo');
        });
        
        it('should handle no results', async () => {
            mockDatabase.search.mockResolvedValue([]);
            
            const results = await service.searchCapitals('Atlantis');
            
            expect(results).toHaveLength(0);
        });
    });
    
    describe('timezone handling', () => {
        it('should calculate correct UTC offset', () => {
            const london = { latitude: 51.5074, longitude: -0.1278, timezone: 'Europe/London' };
            const tokyo = { latitude: 35.6762, longitude: 139.6503, timezone: 'Asia/Tokyo' };
            
            const londonOffset = service.getTimezoneOffset(london);
            const tokyoOffset = service.getTimezoneOffset(tokyo);
            
            // During standard time
            expect(londonOffset).toBe(0);
            expect(tokyoOffset).toBe(9);
        });
    });
});
```

## 3. Integration Test Specifications

### 3.1 Wallpaper Update Flow Tests

```typescript
describe('Wallpaper Update Integration', () => {
    let moonPhaseManager: MoonPhaseManager;
    let wallpaperService: WallpaperService;
    let platformBridge: PlatformBridge;
    
    beforeEach(async () => {
        // Setup integrated system
        const container = await createTestContainer();
        moonPhaseManager = container.get(MoonPhaseManager);
        wallpaperService = container.get(WallpaperService);
        platformBridge = container.get(PlatformBridge);
        
        // Mock platform operations
        jest.spyOn(platformBridge, 'setWallpaper').mockResolvedValue(true);
    });
    
    it('should complete full update cycle', async () => {
        // Set user preferences
        await moonPhaseManager.setLocation(mockLondonLocation());
        await moonPhaseManager.setTheme('artistic');
        
        // Trigger update
        await wallpaperService.updateWallpaper();
        
        // Verify complete flow
        expect(moonPhaseManager.getCurrentPhase()).toBeDefined();
        expect(platformBridge.setWallpaper).toHaveBeenCalledWith(
            expect.objectContaining({
                width: expect.any(Number),
                height: expect.any(Number)
            })
        );
    });
    
    it('should handle location change correctly', async () => {
        // Initial location
        await moonPhaseManager.setLocation(mockLondonLocation());
        const initialPhase = moonPhaseManager.getCurrentPhase();
        
        // Change location across date line
        await moonPhaseManager.setLocation(mockTokyoLocation());
        await wallpaperService.updateWallpaper();
        
        const newPhase = moonPhaseManager.getCurrentPhase();
        
        // Phase might be different due to timezone
        expect(newPhase.calculatedAt).not.toBe(initialPhase.calculatedAt);
    });
});
```

### 3.2 Theme System Integration Tests

```typescript
describe('Theme System Integration', () => {
    let themeController: ThemeController;
    let renderer: WallpaperRenderer;
    let assetManager: AssetManager;
    
    beforeEach(async () => {
        const container = await createTestContainer();
        themeController = container.get(ThemeController);
        renderer = container.get(WallpaperRenderer);
        assetManager = container.get(AssetManager);
    });
    
    it('should load and apply all themes', async () => {
        const themes = ['realistic', 'artistic', 'minimal'];
        
        for (const themeType of themes) {
            // Load theme
            await themeController.setTheme(themeType);
            const theme = themeController.getCurrentTheme();
            
            // Verify assets loaded
            expect(theme.assets).toBeDefined();
            expect(assetManager.isLoaded(theme.id)).toBe(true);
            
            // Verify rendering works
            const preview = await renderer.previewTheme(theme, MoonPhase.FULL_MOON);
            expect(preview).toBeDefined();
        }
    });
    
    it('should handle theme switching without memory leaks', async () => {
        const initialMemory = process.memoryUsage().heapUsed;
        
        // Switch themes multiple times
        for (let i = 0; i < 10; i++) {
            await themeController.setTheme('realistic');
            await themeController.setTheme('artistic');
            await themeController.setTheme('minimal');
        }
        
        // Force garbage collection
        global.gc();
        
        const finalMemory = process.memoryUsage().heapUsed;
        const memoryIncrease = finalMemory - initialMemory;
        
        // Should not leak more than 10MB
        expect(memoryIncrease).toBeLessThan(10 * 1024 * 1024);
    });
});
```

## 4. End-to-End Test Specifications

### 4.1 User Journey Tests

```typescript
describe('E2E User Journeys', () => {
    let app: Application;
    
    beforeEach(async () => {
        app = await Application.launch({
            platform: 'android',
            clearData: true
        });
    });
    
    afterEach(async () => {
        await app.close();
    });
    
    it('should complete first-time setup', async () => {
        // Launch app
        await app.waitForElement('welcomeScreen');
        
        // Skip or view intro
        await app.tap('getStartedButton');
        
        // Select location
        await app.waitForElement('locationSelector');
        await app.type('searchInput', 'London');
        await app.tap('searchResult_0');
        await app.tap('confirmLocationButton');
        
        // Select theme
        await app.waitForElement('themeSelector');
        await app.tap('artisticTheme');
        await app.tap('confirmThemeButton');
        
        // Verify wallpaper set
        await app.waitForElement('successMessage');
        const wallpaperSet = await app.platform.isWallpaperSet();
        expect(wallpaperSet).toBe(true);
    });
    
    it('should update wallpaper at midnight', async () => {
        // Setup app with preferences
        await setupAppWithDefaults(app);
        
        // Get current wallpaper
        const initialWallpaper = await app.platform.getCurrentWallpaper();
        
        // Simulate time change to next day
        await app.platform.setSystemTime('2024-01-15 23:59:00');
        await app.wait(2 * 60 * 1000); // Wait 2 minutes
        
        // Verify wallpaper updated
        const newWallpaper = await app.platform.getCurrentWallpaper();
        expect(newWallpaper).not.toEqual(initialWallpaper);
    });
});
```

### 4.2 Error Scenario Tests

```typescript
describe('E2E Error Handling', () => {
    it('should handle permission denial gracefully', async () => {
        const app = await Application.launch({ platform: 'android' });
        
        // Deny wallpaper permission
        await app.platform.denyPermission('SET_WALLPAPER');
        
        // Try to set wallpaper
        await setupAppWithDefaults(app);
        
        // Should show permission explanation
        await app.waitForElement('permissionExplanation');
        await app.tap('openSettingsButton');
        
        // Verify redirected to settings
        expect(await app.platform.isInSettings()).toBe(true);
    });
    
    it('should work offline after initial setup', async () => {
        const app = await Application.launch({ platform: 'ios' });
        
        // Complete setup online
        await setupAppWithDefaults(app);
        
        // Go offline
        await app.platform.setNetworkState('offline');
        
        // Should still calculate moon phases
        await app.open();
        await app.tap('refreshButton');
        
        // Verify moon phase displayed
        await app.waitForElement('currentMoonPhase');
        const phaseText = await app.getText('moonPhaseLabel');
        expect(phaseText).toMatch(/Moon|Crescent|Quarter|Gibbous/);
    });
});
```

## 5. Performance Test Specifications

### 5.1 Rendering Performance Tests

```typescript
describe('Performance Tests', () => {
    describe('Rendering Performance', () => {
        it('should render wallpaper within 2 seconds', async () => {
            const renderer = new WallpaperRenderer();
            const moonData = createMockMoonData(MoonPhase.FULL_MOON);
            const theme = await loadTheme('realistic');
            const dimensions = { width: 1440, height: 3200 }; // High res
            
            const startTime = performance.now();
            await renderer.render(moonData, theme, dimensions);
            const endTime = performance.now();
            
            expect(endTime - startTime).toBeLessThan(2000);
        });
        
        it('should handle multiple concurrent renders', async () => {
            const renderer = new WallpaperRenderer();
            const renderTasks = [];
            
            // Queue 5 concurrent renders
            for (let i = 0; i < 5; i++) {
                const task = renderer.render(
                    createMockMoonData(MoonPhase.FULL_MOON),
                    await loadTheme('artistic'),
                    { width: 1080, height: 1920 }
                );
                renderTasks.push(task);
            }
            
            const startTime = performance.now();
            await Promise.all(renderTasks);
            const endTime = performance.now();
            
            // Should complete all within reasonable time
            expect(endTime - startTime).toBeLessThan(5000);
        });
    });
    
    describe('Memory Usage', () => {
        it('should not exceed 100MB memory usage', async () => {
            const app = new Application();
            await app.initialize();
            
            // Perform typical operations
            await app.setLocation(mockLondonLocation());
            await app.setTheme('realistic');
            await app.updateWallpaper();
            
            const memoryUsage = await app.getMemoryUsage();
            expect(memoryUsage.total).toBeLessThan(100 * 1024 * 1024);
        });
    });
});
```

## 6. Platform-Specific Tests

### 6.1 Android-Specific Tests

```typescript
describe('Android Platform Tests', () => {
    let androidBridge: AndroidPlatform;
    
    beforeEach(() => {
        androidBridge = new AndroidPlatform();
    });
    
    it('should set wallpaper using WallpaperManager', async () => {
        const bitmap = createTestBitmap(1080, 1920);
        const mockWallpaperManager = createMockWallpaperManager();
        
        androidBridge.wallpaperManager = mockWallpaperManager;
        
        await androidBridge.setWallpaper(bitmap);
        
        expect(mockWallpaperManager.setBitmap).toHaveBeenCalledWith(bitmap);
    });
    
    it('should schedule background updates with WorkManager', async () => {
        const mockWorkManager = createMockWorkManager();
        androidBridge.workManager = mockWorkManager;
        
        await androidBridge.scheduleBackgroundTask({
            id: 'moon_phase_update',
            interval: 24 * 60 * 60 * 1000, // Daily
            constraints: {
                requiresCharging: false,
                requiresNetwork: false
            }
        });
        
        expect(mockWorkManager.enqueueUniquePeriodicWork).toHaveBeenCalled();
    });
});
```

### 6.2 iOS-Specific Tests

```typescript
describe('iOS Platform Tests', () => {
    let iosBridge: IOSPlatform;
    
    beforeEach(() => {
        iosBridge = new IOSPlatform();
    });
    
    it('should save wallpaper to Photos app', async () => {
        const image = createTestImage(1170, 2532); // iPhone 12 Pro
        const mockPhotoLibrary = createMockPhotoLibrary();
        
        iosBridge.photoLibrary = mockPhotoLibrary;
        
        await iosBridge.setWallpaper(image);
        
        expect(mockPhotoLibrary.saveImage).toHaveBeenCalledWith(
            image,
            expect.objectContaining({
                album: 'Moon Phase Wallpapers'
            })
        );
    });
    
    it('should show wallpaper instructions after save', async () => {
        const image = createTestImage(1170, 2532);
        const mockUI = createMockUIKit();
        
        iosBridge.uiKit = mockUI;
        
        await iosBridge.setWallpaper(image);
        
        expect(mockUI.showAlert).toHaveBeenCalledWith(
            expect.objectContaining({
                title: 'Wallpaper Saved',
                actions: expect.arrayContaining(['Open Settings'])
            })
        );
    });
});
```

## 7. Test Data Fixtures

### 7.1 Moon Phase Test Data

```typescript
export const moonPhaseTestData = {
    newMoon: {
        date: '2021-01-13T05:00:00Z',
        expectedPhase: MoonPhase.NEW_MOON,
        expectedIllumination: 0
    },
    firstQuarter: {
        date: '2021-01-20T21:01:00Z',
        expectedPhase: MoonPhase.FIRST_QUARTER,
        expectedIllumination: 50
    },
    fullMoon: {
        date: '2021-01-28T19:16:00Z',
        expectedPhase: MoonPhase.FULL_MOON,
        expectedIllumination: 100
    },
    lastQuarter: {
        date: '2021-02-04T17:37:00Z',
        expectedPhase: MoonPhase.LAST_QUARTER,
        expectedIllumination: 50
    }
};
```

### 7.2 Location Test Data

```typescript
export const locationTestData = {
    majorCities: [
        { name: 'London', country: 'UK', lat: 51.5074, lng: -0.1278, tz: 'Europe/London' },
        { name: 'Tokyo', country: 'Japan', lat: 35.6762, lng: 139.6503, tz: 'Asia/Tokyo' },
        { name: 'New York', country: 'USA', lat: 40.7128, lng: -74.0060, tz: 'America/New_York' },
        { name: 'Sydney', country: 'Australia', lat: -33.8688, lng: 151.2093, tz: 'Australia/Sydney' }
    ],
    edgeCases: [
        { name: 'Reykjavik', country: 'Iceland', lat: 64.1466, lng: -21.9426, tz: 'Atlantic/Reykjavik' },
        { name: 'Singapore', country: 'Singapore', lat: 1.3521, lng: 103.8198, tz: 'Asia/Singapore' }
    ]
};
```

## 8. Continuous Integration Setup

### 8.1 Test Execution Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: npm run test:unit
      - name: Upload coverage
        uses: codecov/codecov-action@v2

  integration-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm run test:integration

  e2e-android:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Android emulator
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 29
          script: npm run test:e2e:android

  e2e-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup iOS simulator
        run: xcrun simctl boot "iPhone 12"
      - run: npm run test:e2e:ios
```

### 8.2 Test Reporting

```typescript
// jest.config.js
module.exports = {
    collectCoverage: true,
    coverageDirectory: 'coverage',
    coverageReporters: ['html', 'lcov', 'text'],
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 90,
            lines: 90,
            statements: 90
        }
    },
    reporters: [
        'default',
        ['jest-junit', {
            outputDirectory: 'test-results',
            outputName: 'junit.xml'
        }]
    ]
};
```