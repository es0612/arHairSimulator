# Project Structure & Organization

## Overall Architecture

The project follows Clean Architecture principles with modular SPM packages and XcodeGen project management.

```
ARHairColorSimulator/
├── project.yml                   # XcodeGen configuration
├── App/                          # Main application (Frameworks & Drivers)
├── Packages/                     # SPM local packages
├── Tests/                        # App-level UI tests
├── Scripts/                      # Build and setup scripts
└── .kiro/                        # Kiro configuration and specs
```

## Package Structure (Clean Architecture Layers)

### Domain Package (`Packages/Domain/`)
**Core business logic and entities - no external dependencies**
```
Sources/
├── Entities/                     # Core business objects
│   ├── HairColor.swift          # Color models and categories
│   ├── User.swift               # User and profile models
│   └── SavedSimulation.swift    # Simulation data models
├── UseCases/                     # Application business rules
│   ├── ApplyHairColorUseCase.swift
│   ├── ColorSelectionUseCase.swift
│   └── CompareHairColorUseCase.swift
├── Repositories/                 # Repository interfaces
│   ├── UserRepositoryProtocol.swift
│   └── SimulationRepositoryProtocol.swift
└── Services/                     # Domain services
    └── ColorProcessingService.swift
```

### Infrastructure Package (`Packages/Infrastructure/`)
**External frameworks and data access implementations**
```
Sources/
├── Persistence/                  # Data storage implementations
│   ├── SwiftDataRepository.swift
│   └── FirebaseRepository.swift
├── AR/                          # ARKit implementations
│   ├── ARCameraManager.swift
│   └── HairSegmentationService.swift
├── Network/                     # Firebase SDK integrations
│   ├── AuthenticationService.swift
│   └── CloudSyncService.swift
└── External/                    # Third-party integrations
    └── SocialSharingService.swift
```

### Presentation Package (`Packages/Presentation/`)
**ViewModels and presentation logic**
```
Sources/
├── ViewModels/                  # @Observable ViewModels
│   ├── CameraViewModel.swift
│   ├── ColorPaletteViewModel.swift
│   └── HistoryViewModel.swift
└── Presenters/                  # Presentation logic
    └── SimulationPresenter.swift
```

## App Structure (`App/`)

### Main Application
```
Sources/
├── Views/                       # SwiftUI Views
│   ├── Camera/
│   │   ├── CameraView.swift
│   │   └── ARCameraPreview.swift
│   ├── ColorPalette/
│   │   ├── ColorPaletteView.swift
│   │   └── ColorAdjustmentView.swift
│   ├── History/
│   │   ├── HistoryView.swift
│   │   └── SimulationDetailView.swift
│   └── Onboarding/
│       ├── OnboardingView.swift
│       └── TutorialView.swift
├── Adapters/                    # Interface adapters
│   ├── FirebaseAdapter.swift
│   └── ARKitAdapter.swift
└── Resources/                   # Assets and localization
    ├── Assets.xcassets
    ├── Localizable.strings (ja)
    └── Localizable.strings (en)
```

## Configuration Files

### XcodeGen Configuration (`project.yml`)
- Defines targets, dependencies, and build settings
- Manages Firebase package dependencies
- Configures local SPM package relationships
- Sets up test schemes and CI/CD integration

### Build Scripts (`Scripts/`)
- `generate_project.sh`: XcodeGen execution and project setup
- `setup_dependencies.sh`: Dependency resolution and package testing
- `clean_build.sh`: Clean build and cache management

## Kiro Integration (`.kiro/`)

### Specifications
```
specs/ar-hair-color-simulator/
├── requirements.md              # Detailed requirements (Japanese)
├── design.md                   # Technical design document
└── tasks.md                    # Implementation roadmap
```

### Steering Rules
```
steering/
├── product.md                  # Product overview and business context
├── tech.md                     # Technology stack and build commands
└── structure.md               # This file - project organization
```

## Development Workflow

### File Organization Principles
1. **Separation of Concerns**: Each package has a single responsibility
2. **Dependency Direction**: Outer layers depend on inner layers only
3. **Testability**: Business logic isolated in Domain package for fast testing
4. **Modularity**: Features can be developed and tested independently

### Naming Conventions
- **Files**: PascalCase for Swift files (`HairColor.swift`)
- **Directories**: PascalCase for feature groupings (`ColorPalette/`)
- **Packages**: PascalCase (`Domain`, `Infrastructure`, `Presentation`)
- **Tests**: Suffix with `Tests` (`DomainTests`, `InfrastructureTests`)

### Import Guidelines
- Domain package: No external imports except Foundation
- Infrastructure: Can import Domain + external frameworks
- Presentation: Can import Domain + SwiftUI
- App: Can import all packages + system frameworks

### Testing Structure
Each package contains its own `Tests/` directory with Swift Testing framework. App-level UI tests use XCTest for device-specific functionality.