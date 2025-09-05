# Technology Stack & Build System

## Platform & Requirements

- **Platform**: iOS 15.0+
- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI with @Observable state management
- **Architecture**: Clean Architecture with MVVM presentation layer

## Project Management

- **Build System**: XcodeGen for project file generation
- **Package Management**: Swift Package Manager (SPM) with local packages
- **Project Structure**: Modular architecture using SPM local packages

## Core Technologies

### AR & Computer Vision
- **ARKit**: Face tracking and depth information
- **Vision Framework**: Hair segmentation and face detection
- **Core ML**: Custom hair quality recognition models
- **RealityKit**: 3D rendering (when needed)
- **AVFoundation**: Camera control and media capture

### Backend & Data
- **Firebase Authentication**: Apple ID, Google, email authentication
- **Cloud Firestore**: User data and simulation history
- **Firebase Storage**: Image and video file storage
- **Firebase Analytics**: Privacy-compliant usage analytics
- **SwiftData**: Modern local data persistence
- **UserDefaults**: Settings and preferences

### Testing & Quality
- **Swift Testing**: Modern testing framework for business logic
- **XCTest**: UI tests and performance tests (legacy support)
- **Test Strategy**: TDD with outside-in approach

## Common Build Commands

### Project Setup
```bash
# Generate Xcode project
./Scripts/generate_project.sh

# Setup dependencies
./Scripts/setup_dependencies.sh

# Clean build
./Scripts/clean_build.sh
```

### Development Workflow
```bash
# Test individual packages (fast)
cd Packages/Domain && swift test
cd Packages/Infrastructure && swift test
cd Packages/Presentation && swift test

# Test entire project
xcodebuild test -project ARHairColorSimulator.xcodeproj -scheme ARHairColorSimulator

# Build for device
xcodebuild -project ARHairColorSimulator.xcodeproj -scheme ARHairColorSimulator -destination 'generic/platform=iOS'
```

### Package Management
```bash
# Resolve package dependencies
xcodebuild -resolvePackageDependencies

# Update packages
xcodebuild -project ARHairColorSimulator.xcodeproj -scheme ARHairColorSimulator -resolvePackageDependencies
```

## Performance Requirements

- **AR Processing**: Maintain 30+ fps during real-time hair color application
- **App Launch**: Start within 1 second on supported devices
- **Memory Management**: Proper CVPixelBuffer cleanup and cache management
- **Battery Optimization**: Efficient AR session management

## Development Guidelines

- Use XcodeGen for all project configuration changes
- Write tests first using Swift Testing framework
- Keep business logic in SPM packages for fast testing
- All AR processing must be device-local for privacy
- Follow Clean Architecture dependency rules
- Use @Observable for SwiftUI state management