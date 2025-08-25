# 設計文書

## 概要

ARヘアカラーシミュレーターは、ARKit、Vision Framework、SwiftUI、Firebaseを活用したiOSアプリケーションです。リアルタイムで髪の色を変更し、自然なシミュレーション体験を提供します。アプリは落ち着いたデザインと高いパフォーマンスを重視し、プライバシーファーストのアプローチを採用します。

## アーキテクチャ

### 全体アーキテクチャ

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   SwiftUI Views │    │  AR Processing  │    │ Firebase Cloud  │
│                 │    │                 │    │                 │
│ • Camera View   │◄──►│ • ARKit         │    │ • Authentication│
│ • Color Picker  │    │ • Vision        │    │ • Firestore     │
│ • History       │    │ • Core ML       │    │ • Storage       │
│ • Settings      │    │ • RealityKit    │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │ Local Storage   │
                    │                 │
                    │ • SwiftData     │
                    │ • UserDefaults  │
                    │ • File System   │
                    └─────────────────┘
```

### SPMローカルパッケージ構造

```
ARHairColorSimulator/
├── App/                          # メインアプリケーション
│   ├── Views/                    # SwiftUI Views
│   ├── ViewModels/               # @Observable ViewModels
│   └── ARHairColorSimulatorApp.swift
├── Packages/                     # ローカルSPMパッケージ
│   ├── HairColorCore/            # コアビジネスロジック
│   │   ├── Sources/
│   │   │   ├── Models/
│   │   │   ├── Services/
│   │   │   └── Utilities/
│   │   └── Tests/                # Swift Testing
│   ├── ARProcessing/             # AR関連処理
│   │   ├── Sources/
│   │   └── Tests/
│   ├── ColorEngine/              # 色処理エンジン
│   │   ├── Sources/
│   │   └── Tests/
│   └── NetworkLayer/             # Firebase通信
│       ├── Sources/
│       └── Tests/
└── Tests/                        # アプリレベルのUIテスト
```

### レイヤー構造

1. **プレゼンテーション層 (SwiftUI)**
   - ユーザーインターフェース
   - @Observable ViewModels
   - ナビゲーション管理

2. **ビジネスロジック層 (SPMパッケージ)**
   - HairColorCore: コアビジネスロジック
   - ColorEngine: 色処理とカスタマイズ
   - ARProcessing: AR関連処理
   - NetworkLayer: Firebase通信

3. **データ層**
   - ローカルストレージ (SwiftData)
   - クラウドストレージ (Firebase)
   - キャッシュ管理

4. **AR処理層**
   - 顔検出とトラッキング
   - 髪のセグメンテーション
   - リアルタイム色適用

## コンポーネントとインターフェース

### 主要コンポーネント

#### 1. ARCameraManager
```swift
@Observable
class ARCameraManager {
    var isSessionRunning: Bool = false
    var faceDetected: Bool = false
    var hairMask: CVPixelBuffer?
    
    func startSession()
    func stopSession()
    func applyHairColor(_ color: HairColor)
    func capturePhoto() -> UIImage?
    func captureVideo() -> URL?
}
```

#### 2. HairSegmentationService
```swift
protocol HairSegmentationService {
    func segmentHair(from pixelBuffer: CVPixelBuffer) async -> CVPixelBuffer?
    func updateSegmentation(with faceGeometry: ARFaceGeometry)
}

class VisionHairSegmentationService: HairSegmentationService {
    private let visionModel: VNCoreMLModel
    private let depthProcessor: DepthProcessor
}
```

#### 3. ColorPaletteManager
```swift
@Observable
class ColorPaletteManager {
    var availableColors: [HairColor] = []
    var selectedColor: HairColor?
    
    func loadColors()
    func createCustomColor(hue: Double, saturation: Double, brightness: Double) -> HairColor
    func applyColorAdjustments(_ adjustments: ColorAdjustments) -> HairColor
}
```

#### 4. UserManager
```swift
@Observable
class UserManager {
    var currentUser: User?
    var isAuthenticated: Bool = false
    
    func signIn(with provider: AuthProvider) async throws
    func signOut() async throws
    func updateProfile(_ profile: UserProfile) async throws
}
```

#### 5. SimulationHistoryManager
```swift
@Observable
class SimulationHistoryManager {
    var savedSimulations: [SavedSimulation] = []
    
    func saveSimulation(_ simulation: HairSimulation) async throws
    func loadHistory() async throws
    func deleteSimulation(_ id: UUID) async throws
    func shareSimulation(_ simulation: SavedSimulation) -> ShareItem
}
```

### データモデル

#### HairColor
```swift
struct HairColor: Codable, Identifiable {
    let id: UUID
    let name: String
    let baseColor: Color
    let hue: Double
    let saturation: Double
    let brightness: Double
    let category: ColorCategory
    let isGradient: Bool
    let gradientColors: [Color]?
}

enum ColorCategory: String, CaseIterable {
    case natural = "ナチュラル"
    case fashion = "ファッション"
    case pastel = "パステル"
    case vivid = "ビビッド"
    case gradient = "グラデーション"
}
```

#### User
```swift
struct User: Codable, Identifiable {
    let id: String
    var email: String?
    var displayName: String?
    var profile: UserProfile
    var preferences: UserPreferences
    var createdAt: Date
    var lastLoginAt: Date
}

struct UserProfile: Codable {
    var age: Int?
    var gender: Gender?
    var hairType: HairType
    var skinTone: SkinTone?
    var faceShape: FaceShape?
}

enum HairType: String, CaseIterable {
    case straight = "ストレート"
    case wavy = "ウェーブ"
    case curly = "カール"
}
```

#### SavedSimulation (SwiftData Model)
```swift
import SwiftData

@Model
class SavedSimulation {
    @Attribute(.unique) var id: UUID
    var userId: String
    var hairColor: HairColor
    var originalImage: Data
    var simulatedImage: Data
    var videoData: Data?
    var createdAt: Date
    var tags: [String]
    var isFavorite: Bool
    
    init(id: UUID = UUID(), userId: String, hairColor: HairColor, originalImage: Data, simulatedImage: Data, videoData: Data? = nil, createdAt: Date = Date(), tags: [String] = [], isFavorite: Bool = false) {
        self.id = id
        self.userId = userId
        self.hairColor = hairColor
        self.originalImage = originalImage
        self.simulatedImage = simulatedImage
        self.videoData = videoData
        self.createdAt = createdAt
        self.tags = tags
        self.isFavorite = isFavorite
    }
}
```

## エラーハンドリング

### エラータイプ定義
```swift
enum ARHairSimulatorError: LocalizedError {
    case cameraPermissionDenied
    case arSessionFailed(ARError)
    case faceDetectionFailed
    case hairSegmentationFailed
    case authenticationFailed(AuthError)
    case networkError(Error)
    case storageError(Error)
    
    var errorDescription: String? {
        switch self {
        case .cameraPermissionDenied:
            return "カメラへのアクセス許可が必要です"
        case .arSessionFailed(let error):
            return "ARセッションエラー: \(error.localizedDescription)"
        case .faceDetectionFailed:
            return "顔を検出できませんでした。明るい場所で正面を向いてください"
        case .hairSegmentationFailed:
            return "髪の検出に失敗しました。髪が見えるように調整してください"
        case .authenticationFailed(let error):
            return "認証エラー: \(error.localizedDescription)"
        case .networkError(let error):
            return "ネットワークエラー: \(error.localizedDescription)"
        case .storageError(let error):
            return "保存エラー: \(error.localizedDescription)"
        }
    }
}
```

### エラーハンドリング戦略
1. **ユーザーフレンドリーなメッセージ**: 技術的な詳細を隠し、解決策を提示
2. **自動復旧**: 可能な場合は自動的にリトライ
3. **グレースフルデグラデーション**: 機能の一部が利用できない場合の代替手段
4. **ログ記録**: デバッグ用の詳細ログ（個人情報は除外）

## テスト戦略

### 単体テスト (Swift Testing)
- **HairSegmentationService**: テスト画像を使用した精度テスト
- **ColorPaletteManager**: 色変換ロジックのテスト
- **UserManager**: Firebase認証のモックテスト
- **ビジネスロジック**: SPMローカルパッケージでの高速テスト

### 統合テスト
- **AR + 色適用**: 実際のデバイスでのエンドツーエンドテスト
- **Firebase同期**: ネットワーク状態を変更したテスト
- **オフライン機能**: ネットワーク切断時の動作テスト

### UIテスト (XCTest)
- **アクセシビリティ**: VoiceOverとの互換性テスト
- **パフォーマンス**: フレームレートとメモリ使用量の測定
- **ユーザビリティ**: 実際のユーザーによるテスト

### テスト環境 (Swift Testing)
```swift
import Testing
import Foundation

@Test("Hair color conversion accuracy")
func testHairColorConversion() async throws {
    let colorManager = ColorPaletteManager()
    let testColor = HairColor(name: "Test", baseColor: .red, hue: 0.0, saturation: 1.0, brightness: 1.0, category: .fashion)
    
    let convertedColor = colorManager.applyColorAdjustments(ColorAdjustments(brightness: 0.8))
    
    #expect(convertedColor.brightness == 0.8)
}

@Test("User authentication flow")
func testUserAuthentication() async throws {
    let mockAuth = MockFirebaseAuth()
    let userManager = UserManager(authService: mockAuth)
    
    try await userManager.signIn(with: .email("test@example.com", "password"))
    
    #expect(userManager.isAuthenticated == true)
    #expect(userManager.currentUser != nil)
}

// モック実装
class MockFirebaseAuth: AuthService {
    var mockUser: User?
    
    func signIn(withEmail email: String, password: String) async throws -> AuthDataResult {
        // モック実装
        return AuthDataResult(user: mockUser ?? User.mock)
    }
}
```

## 技術スタック詳細

### フロントエンド (SwiftUI)
- **SwiftUI**: 宣言的UI構築
- **Observation**: @Observable マクロによるモダンな状態管理
- **Swift Charts**: カラーパレット表示
- **AVFoundation**: カメラ制御

### AR/ML処理
- **ARKit**: 顔トラッキングと深度情報
- **Vision Framework**: 髪のセグメンテーション
- **Core ML**: カスタム髪質認識モデル
- **RealityKit**: 3Dレンダリング（必要に応じて）

### バックエンド (Firebase)
- **Firebase Authentication**: ユーザー認証
  - Apple ID、Google、メール認証
  - ゲストモード対応
- **Cloud Firestore**: ユーザーデータとシミュレーション履歴
- **Firebase Storage**: 画像・動画ファイル保存
- **Firebase Analytics**: 使用状況分析（プライバシー準拠）

### ローカルストレージ
- **SwiftData**: モダンなデータ永続化フレームワーク
- **UserDefaults**: 設定とプリファレンス
- **FileManager**: 一時ファイルとキャッシュ

### テスティング
- **Swift Testing**: モダンなテストフレームワーク
- **XCTest**: UIテストとパフォーマンステスト（併用）

### ネットワーキング
- **URLSession**: 基本的なHTTP通信
- **Firebase SDK**: Firebase サービスとの通信
- **NetworkLayer Package**: SPMローカルパッケージでの抽象化

## パフォーマンス最適化

### AR処理最適化
1. **フレームレート管理**: 30fps以上を維持
2. **メモリ管理**: CVPixelBufferの適切な解放
3. **バックグラウンド処理**: メインスレッドをブロックしない
4. **適応的品質**: デバイス性能に応じた処理品質調整

### UI最適化
1. **遅延読み込み**: 大きな画像の非同期読み込み
2. **キャッシュ戦略**: 頻繁に使用される色とテクスチャのキャッシュ
3. **アニメーション最適化**: 60fpsでのスムーズなトランジション

### ストレージ最適化
1. **画像圧縮**: 適切な品質での画像保存
2. **データ同期**: 差分同期によるネットワーク使用量削減
3. **キャッシュ管理**: LRUキャッシュによるメモリ効率化

## セキュリティとプライバシー

### データ保護
1. **ローカル処理**: 顔データはデバイス内で処理
2. **暗号化**: 保存データの暗号化
3. **匿名化**: 分析データの個人情報除去
4. **GDPR準拠**: ユーザーの同意管理とデータ削除権

### 認証セキュリティ
1. **OAuth 2.0**: 標準的な認証プロトコル
2. **トークン管理**: 安全なトークン保存と更新
3. **セッション管理**: 適切なセッションタイムアウト

### アプリセキュリティ
1. **コード難読化**: リリースビルドでの難読化
2. **証明書ピニング**: HTTPS通信の追加保護
3. **ジェイルブレイク検出**: 不正な環境での実行防止

## 国際化とアクセシビリティ

### 多言語対応
- **日本語**: プライマリ言語
- **英語**: セカンダリ言語
- **Localizable.strings**: 文字列の外部化
- **右から左への言語**: 将来的な対応準備

### アクセシビリティ
- **VoiceOver**: 完全対応
- **Dynamic Type**: フォントサイズ調整
- **高コントラスト**: 視覚障害者向け表示
- **音声ガイド**: AR操作の音声フィードバック

## 展開とメンテナンス

### CI/CD パイプライン
1. **自動テスト**: プルリクエスト時の自動テスト実行
2. **コード品質**: SwiftLintによる静的解析
3. **自動ビルド**: TestFlightへの自動配布
4. **パフォーマンス監視**: Firebase Performanceによる監視

### 監視とログ
1. **クラッシュレポート**: Firebase Crashlyticsによる監視
2. **パフォーマンス監視**: フレームレートとメモリ使用量
3. **ユーザー行動分析**: プライバシーに配慮した使用状況分析
4. **エラー追跡**: 詳細なエラーログとスタックトレース

### アップデート戦略
1. **段階的ロールアウト**: 新機能の段階的リリース
2. **A/Bテスト**: UI/UX改善のためのテスト
3. **後方互換性**: 古いバージョンとの互換性維持
4. **緊急修正**: 重要なバグの迅速な修正
##
 SPMローカルパッケージ戦略の利点

### 開発効率の向上
1. **高速テスト実行**: `swift test` による高速なユニットテスト
2. **TDD対応**: ビジネスロジックの迅速なテストサイクル
3. **モジュール化**: 関心の分離による保守性向上
4. **並行開発**: パッケージ単位での独立開発

### コード品質の向上
1. **依存関係の明確化**: パッケージ間の依存関係が明示的
2. **再利用性**: 他のプロジェクトでの再利用可能
3. **テスタビリティ**: シミュレータ依存のないロジックの分離
4. **型安全性**: Swift Package Managerによる型チェック

### パッケージ構成詳細

#### HairColorCore Package
```swift
// Package.swift
let package = Package(
    name: "HairColorCore",
    platforms: [.iOS(.v15)],
    products: [
        .library(name: "HairColorCore", targets: ["HairColorCore"])
    ],
    targets: [
        .target(name: "HairColorCore"),
        .testTarget(name: "HairColorCoreTests", dependencies: ["HairColorCore"])
    ]
)
```

#### ColorEngine Package
```swift
// Package.swift
let package = Package(
    name: "ColorEngine",
    platforms: [.iOS(.v15)],
    products: [
        .library(name: "ColorEngine", targets: ["ColorEngine"])
    ],
    dependencies: [
        .package(path: "../HairColorCore")
    ],
    targets: [
        .target(name: "ColorEngine", dependencies: ["HairColorCore"]),
        .testTarget(name: "ColorEngineTests", dependencies: ["ColorEngine"])
    ]
)
```

### テスト戦略の最適化

#### Swift Testing の活用
```swift
import Testing
import HairColorCore

@Suite("Hair Color Processing")
struct HairColorTests {
    
    @Test("Color conversion maintains hue accuracy")
    func colorConversionAccuracy() async throws {
        let originalColor = HairColor.natural(.brown)
        let convertedColor = try await ColorEngine.convert(originalColor, to: .fashion(.red))
        
        #expect(convertedColor.category == .fashion)
        #expect(abs(convertedColor.hue - 0.0) < 0.01) // Red hue
    }
    
    @Test("Brightness adjustment preserves saturation", arguments: [0.2, 0.5, 0.8, 1.0])
    func brightnessAdjustment(brightness: Double) async throws {
        let originalColor = HairColor.fashion(.blue)
        let adjustedColor = ColorEngine.adjustBrightness(originalColor, to: brightness)
        
        #expect(adjustedColor.brightness == brightness)
        #expect(adjustedColor.saturation == originalColor.saturation)
    }
}
```

#### パフォーマンステスト
```swift
@Test("Color processing performance")
func colorProcessingPerformance() async throws {
    let colors = Array(repeating: HairColor.natural(.black), count: 1000)
    
    let startTime = ContinuousClock.now
    let processedColors = try await ColorEngine.batchProcess(colors)
    let duration = ContinuousClock.now - startTime
    
    #expect(processedColors.count == 1000)
    #expect(duration < .milliseconds(100)) // 100ms以内で処理完了
}
```

### CI/CD最適化

#### GitHub Actions での並列テスト
```yaml
name: Test Packages
on: [push, pull_request]

jobs:
  test-packages:
    runs-on: macos-latest
    strategy:
      matrix:
        package: [HairColorCore, ColorEngine, ARProcessing, NetworkLayer]
    steps:
      - uses: actions/checkout@v3
      - name: Test ${{ matrix.package }}
        run: |
          cd Packages/${{ matrix.package }}
          swift test --parallel
```

この戦略により、開発効率が大幅に向上し、高品質なコードベースを維持できます。## クリー
ンアーキテクチャ設計

### レイヤー構造の詳細化

```
┌─────────────────────────────────────────────────────────┐
│                    Frameworks & Drivers                 │
│  SwiftUI Views, ARKit, Firebase SDK, SwiftData         │
└─────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────┐
│                Interface Adapters                       │
│  ViewModels, Presenters, Controllers, Gateways         │
└─────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────┐
│                   Use Cases                             │
│  Application Business Rules                             │
└─────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────┐
│                    Entities                             │
│  Enterprise Business Rules                              │
└─────────────────────────────────────────────────────────┘
```

### パッケージ構造の再設計

```
ARHairColorSimulator/
├── App/                          # Frameworks & Drivers
│   ├── Views/                    # SwiftUI Views
│   ├── ViewModels/               # Interface Adapters
│   └── Adapters/                 # External Service Adapters
├── Packages/
│   ├── Domain/                   # Entities + Use Cases
│   │   ├── Sources/
│   │   │   ├── Entities/         # Core Business Objects
│   │   │   ├── UseCases/         # Application Business Rules
│   │   │   ├── Repositories/     # Repository Interfaces
│   │   │   └── Services/         # Domain Services
│   │   └── Tests/
│   ├── Infrastructure/           # Frameworks & Drivers
│   │   ├── Sources/
│   │   │   ├── Persistence/      # SwiftData, Firebase
│   │   │   ├── AR/               # ARKit Implementation
│   │   │   └── Network/          # Firebase SDK
│   │   └── Tests/
│   └── Presentation/             # Interface Adapters
│       ├── Sources/
│       │   ├── ViewModels/
│       │   └── Presenters/
│       └── Tests/
```

## t-wada流TDD戦略

### テスト駆動開発のサイクル

#### 1. Red-Green-Refactor サイクル
```swift
// 1. Red: 失敗するテストを書く
@Test("Hair color should be applied to detected hair region")
func applyHairColorToDetectedRegion() async throws {
    let useCase = ApplyHairColorUseCase(
        hairDetector: MockHairDetector(),
        colorProcessor: MockColorProcessor()
    )
    
    let result = try await useCase.execute(
        image: TestImages.faceWithHair,
        color: HairColor.natural(.brown)
    )
    
    #expect(result.isSuccess)
    #expect(result.processedImage != nil)
}

// 2. Green: 最小限の実装で通す
class ApplyHairColorUseCase {
    func execute(image: UIImage, color: HairColor) async throws -> HairColorResult {
        return HairColorResult(isSuccess: true, processedImage: image)
    }
}

// 3. Refactor: 実装を改善
class ApplyHairColorUseCase {
    private let hairDetector: HairDetectorProtocol
    private let colorProcessor: ColorProcessorProtocol
    
    func execute(image: UIImage, color: HairColor) async throws -> HairColorResult {
        let hairMask = try await hairDetector.detectHair(in: image)
        let processedImage = try await colorProcessor.applyColor(color, to: image, mask: hairMask)
        return HairColorResult(isSuccess: true, processedImage: processedImage)
    }
}
```

#### 2. テストファースト設計
```swift
// まずインターフェースをテストで定義
@Test("Hair detection should identify hair regions accurately")
func hairDetectionAccuracy() async throws {
    let detector = VisionHairDetector()
    let testImage = TestImages.profileWithClearHair
    
    let result = try await detector.detectHair(in: testImage)
    
    #expect(result.confidence > 0.8)
    #expect(result.hairRegion.area > 0)
}

// インターフェースを定義
protocol HairDetectorProtocol {
    func detectHair(in image: UIImage) async throws -> HairDetectionResult
}

struct HairDetectionResult {
    let confidence: Double
    let hairRegion: CGRect
    let mask: CVPixelBuffer
}
```

#### 3. 境界値テスト
```swift
@Suite("Hair Color Boundary Tests")
struct HairColorBoundaryTests {
    
    @Test("Extreme brightness values", arguments: [0.0, 1.0])
    func extremeBrightnessValues(brightness: Double) async throws {
        let color = HairColor.custom(hue: 0.5, saturation: 0.5, brightness: brightness)
        let processor = ColorProcessor()
        
        let result = try await processor.validateColor(color)
        
        #expect(result.isValid)
        #expect(result.adjustedColor.brightness >= 0.0)
        #expect(result.adjustedColor.brightness <= 1.0)
    }
    
    @Test("Invalid image inputs")
    func invalidImageInputs() async throws {
        let detector = VisionHairDetector()
        
        await #expect(throws: HairDetectionError.invalidImage) {
            try await detector.detectHair(in: UIImage())
        }
    }
}
```

## Kent Beck の Tidy First 戦略

### 1. 小さな整理から始める

#### Before: 複雑な関数
```swift
func processHairColor(_ image: UIImage, _ color: HairColor, _ settings: ProcessingSettings) async throws -> UIImage {
    // 100行の複雑な処理...
    let faceRect = detectFace(image)
    let hairMask = segmentHair(image, faceRect)
    let adjustedColor = adjustColor(color, settings.brightness, settings.saturation)
    let blendedImage = blendColor(image, hairMask, adjustedColor, settings.blendMode)
    let finalImage = applyPostProcessing(blendedImage, settings.smoothing, settings.edgeEnhancement)
    return finalImage
}
```

#### After: 整理された関数
```swift
func processHairColor(_ image: UIImage, _ color: HairColor, _ settings: ProcessingSettings) async throws -> UIImage {
    let faceRect = try await detectFace(in: image)
    let hairMask = try await segmentHair(in: image, faceRect: faceRect)
    let processedColor = adjustColor(color, with: settings)
    let blendedImage = try await blendColor(image, mask: hairMask, color: processedColor, mode: settings.blendMode)
    return try await applyPostProcessing(to: blendedImage, settings: settings)
}

private func adjustColor(_ color: HairColor, with settings: ProcessingSettings) -> HairColor {
    return color
        .adjustingBrightness(settings.brightness)
        .adjustingSaturation(settings.saturation)
}

private func applyPostProcessing(to image: UIImage, settings: ProcessingSettings) async throws -> UIImage {
    return try await image
        .applySmoothing(settings.smoothing)
        .enhanceEdges(settings.edgeEnhancement)
}
```

### 2. 段階的なリファクタリング

#### Phase 1: Extract Method
```swift
// Before
class HairColorProcessor {
    func process(_ image: UIImage) async throws -> UIImage {
        // 複雑な処理が混在
        let visionRequest = VNDetectFaceRectanglesRequest()
        let handler = VNImageRequestHandler(cgImage: image.cgImage!)
        try handler.perform([visionRequest])
        // ... 50行の処理
    }
}

// After: メソッド抽出
class HairColorProcessor {
    func process(_ image: UIImage) async throws -> UIImage {
        let faceRect = try await detectFace(in: image)
        let hairMask = try await createHairMask(from: image, faceRect: faceRect)
        return try await applyColorToHair(image, mask: hairMask)
    }
    
    private func detectFace(in image: UIImage) async throws -> CGRect {
        // 顔検出の専用処理
    }
    
    private func createHairMask(from image: UIImage, faceRect: CGRect) async throws -> CVPixelBuffer {
        // 髪マスク作成の専用処理
    }
}
```

#### Phase 2: Extract Class
```swift
// 責任を分離
protocol FaceDetector {
    func detectFace(in image: UIImage) async throws -> FaceDetectionResult
}

protocol HairSegmenter {
    func segmentHair(in image: UIImage, faceRect: CGRect) async throws -> CVPixelBuffer
}

protocol ColorApplicator {
    func applyColor(_ color: HairColor, to image: UIImage, mask: CVPixelBuffer) async throws -> UIImage
}

class HairColorProcessor {
    private let faceDetector: FaceDetector
    private let hairSegmenter: HairSegmenter
    private let colorApplicator: ColorApplicator
    
    func process(_ image: UIImage, color: HairColor) async throws -> UIImage {
        let faceResult = try await faceDetector.detectFace(in: image)
        let hairMask = try await hairSegmenter.segmentHair(in: image, faceRect: faceResult.rect)
        return try await colorApplicator.applyColor(color, to: image, mask: hairMask)
    }
}
```

### 3. テスト可能な設計

#### Dependency Injection
```swift
class HairColorUseCase {
    private let faceDetector: FaceDetector
    private let hairSegmenter: HairSegmenter
    private let colorApplicator: ColorApplicator
    
    init(
        faceDetector: FaceDetector = VisionFaceDetector(),
        hairSegmenter: HairSegmenter = MLHairSegmenter(),
        colorApplicator: ColorApplicator = BlendColorApplicator()
    ) {
        self.faceDetector = faceDetector
        self.hairSegmenter = hairSegmenter
        self.colorApplicator = colorApplicator
    }
}

// テストでのモック使用
@Test("Hair color processing with mocked dependencies")
func hairColorProcessingWithMocks() async throws {
    let useCase = HairColorUseCase(
        faceDetector: MockFaceDetector(),
        hairSegmenter: MockHairSegmenter(),
        colorApplicator: MockColorApplicator()
    )
    
    let result = try await useCase.execute(TestImages.sample, color: .natural(.brown))
    
    #expect(result.isSuccess)
}
```

## 開発ワークフロー統合

### 1. TDD + Clean Architecture サイクル
```
1. 外側から内側へテストを書く (Outside-In TDD)
   - UI Test → Integration Test → Unit Test
   
2. 依存関係の方向を守る
   - 外側のレイヤーが内側のレイヤーに依存
   - インターフェースによる依存性の逆転
   
3. 小さなサイクルで改善
   - Red-Green-Refactor
   - Tidy First の小さな整理
```

### 2. 継続的改善プロセス
```swift
// 週次リファクタリングタスク
enum RefactoringTask {
    case extractMethod(target: String)
    case extractClass(responsibility: String)
    case improveNaming(oldName: String, newName: String)
    case eliminateDuplication(location: String)
    case simplifyConditional(method: String)
}

// リファクタリング前後のメトリクス測定
struct CodeMetrics {
    let cyclomaticComplexity: Int
    let linesOfCode: Int
    let testCoverage: Double
    let duplicatedLines: Int
}
```

この統合された開発戦略により、高品質で保守性の高いコードベースを段階的に構築できます。