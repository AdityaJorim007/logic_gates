# Foundation Models

> **On-device LLMs and transformer models for Apple platforms**

## 🤖 Overview

Foundation models in 2026 run entirely on-device with Apple Silicon optimization.

## Core ML Foundation Models

### 1. Language Models

```swift
import CoreML
import NaturalLanguage

actor LanguageModel {
    private let model: MLModel
    
    init() async throws {
        // Load optimized LLM
        let config = MLModelConfiguration()
        config.computeUnits = .cpuAndNeuralEngine
        
        model = try await MLModel.load(
            contentsOf: Bundle.main.url(
                forResource: "LanguageModel",
                withExtension: "mlmodelc"
            )!,
            configuration: config
        )
    }
    
    func generate(prompt: String, maxTokens: Int = 100) async throws -> String {
        let input = try MLDictionaryFeatureProvider(dictionary: [
            "prompt": prompt,
            "max_tokens": maxTokens
        ])
        
        let output = try await model.prediction(from: input)
        return output.featureValue(for: "generated_text")?.stringValue ?? ""
    }
}

// Usage
let llm = try await LanguageModel()
let response = try await llm.generate(
    prompt: "Explain Swift concurrency:",
    maxTokens: 150
)
```

### 2. Vision Transformers

```swift
import Vision
import CoreML

class ImageAnalyzer {
    private let model: VNCoreMLModel
    
    init() throws {
        let mlModel = try VisionTransformer(configuration: .init()).model
        model = try VNCoreMLModel(for: mlModel)
    }
    
    func analyze(image: UIImage) async throws -> [Classification] {
        guard let cgImage = image.cgImage else {
            throw AnalysisError.invalidImage
        }
        
        let request = VNCoreMLRequest(model: model)
        let handler = VNImageRequestHandler(cgImage: cgImage)
        
        try handler.perform([request])
        
        guard let results = request.results as? [VNClassificationObservation] else {
            return []
        }
        
        return results.map { observation in
            Classification(
                label: observation.identifier,
                confidence: observation.confidence
            )
        }
    }
}

struct Classification {
    let label: String
    let confidence: Float
}
```

### 3. Multimodal Models

```swift
import CoreML

actor MultimodalModel {
    private let model: MLModel
    
    func process(image: UIImage, text: String) async throws -> Response {
        let imageFeature = try encodeImage(image)
        let textFeature = try encodeText(text)
        
        let input = try MLDictionaryFeatureProvider(dictionary: [
            "image": imageFeature,
            "text": textFeature
        ])
        
        let output = try await model.prediction(from: input)
        
        return Response(
            description: output.featureValue(for: "description")?.stringValue ?? "",
            confidence: output.featureValue(for: "confidence")?.doubleValue ?? 0
        )
    }
    
    private func encodeImage(_ image: UIImage) throws -> MLFeatureValue {
        // Image encoding logic
        guard let pixelBuffer = image.pixelBuffer() else {
            throw EncodingError.invalidImage
        }
        return MLFeatureValue(pixelBuffer: pixelBuffer)
    }
    
    private func encodeText(_ text: String) throws -> MLFeatureValue {
        // Text encoding logic
        return MLFeatureValue(string: text)
    }
}

struct Response {
    let description: String
    let confidence: Double
}
```

## 🎯 Optimization Strategies

### 1. Neural Engine Utilization

```swift
let config = MLModelConfiguration()
config.computeUnits = .cpuAndNeuralEngine
config.allowLowPrecisionAccumulationOnGPU = true

let model = try await MLModel.load(
    contentsOf: modelURL,
    configuration: config
)
```

### 2. Quantization

```swift
// Convert model to INT8 quantization
import CoreMLTools

let spec = try MLModel(contentsOf: modelURL).modelDescription
let quantizedModel = try MLModel.compileModel(
    at: modelURL,
    quantizationBits: 8
)
```

### 3. Batch Processing

```swift
actor BatchProcessor {
    private let model: MLModel
    private let batchSize = 32
    
    func process(inputs: [MLFeatureProvider]) async throws -> [MLFeatureProvider] {
        var results: [MLFeatureProvider] = []
        
        for batch in inputs.chunked(into: batchSize) {
            let batchInput = MLArrayBatchProvider(array: batch)
            let batchOutput = try await model.predictions(from: batchInput)
            
            results.append(contentsOf: batchOutput.features)
        }
        
        return results
    }
}
```

## 📱 Platform-Specific Features

### iOS 26

```swift
import CoreML

// On-device training
class PersonalizedModel {
    private var model: MLModel
    
    func personalize(with examples: [TrainingExample]) async throws {
        let updateTask = try MLUpdateTask(
            forModelAt: modelURL,
            trainingData: examples.batchProvider,
            configuration: .init(),
            completionHandler: { context in
                // Handle completion
            }
        )
        
        updateTask.resume()
    }
}
```

### macOS 26

```swift
// Leverage Mac Studio/Pro performance
let config = MLModelConfiguration()
config.computeUnits = .all
config.preferredMetalDevice = MTLCreateSystemDefaultDevice()

let model = try await MLModel.load(
    contentsOf: modelURL,
    configuration: config
)
```

### visionOS 26

```swift
import RealityKit
import CoreML

// Spatial understanding
actor SpatialAnalyzer {
    private let model: MLModel
    
    func analyzeScene(_ scene: Scene) async throws -> SceneUnderstanding {
        let features = extractSpatialFeatures(scene)
        let input = try MLDictionaryFeatureProvider(dictionary: [
            "spatial_features": features
        ])
        
        let output = try await model.prediction(from: input)
        return parseSceneUnderstanding(output)
    }
}
```

## 🔧 Custom Model Integration

### 1. Convert PyTorch to Core ML

```python
import torch
import coremltools as ct

# PyTorch model
model = YourModel()
model.eval()

# Trace model
example_input = torch.rand(1, 3, 224, 224)
traced_model = torch.jit.trace(model, example_input)

# Convert to Core ML
mlmodel = ct.convert(
    traced_model,
    inputs=[ct.TensorType(shape=(1, 3, 224, 224))],
    compute_precision=ct.precision.FLOAT16
)

mlmodel.save("YourModel.mlpackage")
```

### 2. Swift Integration

```swift
import CoreML

class CustomFoundationModel {
    private let model: MLModel
    
    init() async throws {
        model = try await MLModel.load(
            contentsOf: Bundle.main.url(
                forResource: "YourModel",
                withExtension: "mlmodelc"
            )!
        )
    }
    
    func predict(input: MLFeatureProvider) async throws -> MLFeatureProvider {
        try await model.prediction(from: input)
    }
}
```

## 📊 Performance Benchmarks

### Model Sizes (2026)

| Model Type | Size | Inference Time | Platform |
|------------|------|----------------|----------|
| Small LLM | 500MB | 50ms | iPhone 16 Pro |
| Medium LLM | 2GB | 200ms | iPhone 16 Pro |
| Large LLM | 7GB | 800ms | Mac Studio |
| Vision Transformer | 300MB | 30ms | All devices |
| Multimodal | 1.5GB | 150ms | iPhone 16 Pro |

## 🎓 Best Practices

### 1. Model Caching

```swift
actor ModelCache {
    private var models: [String: MLModel] = [:]
    
    func getModel(named name: String) async throws -> MLModel {
        if let cached = models[name] {
            return cached
        }
        
        let model = try await loadModel(named: name)
        models[name] = model
        return model
    }
}
```

### 2. Progressive Loading

```swift
class ProgressiveModelLoader {
    func load(modelName: String, progress: @escaping (Double) -> Void) async throws -> MLModel {
        let url = Bundle.main.url(forResource: modelName, withExtension: "mlmodelc")!
        
        return try await withCheckedThrowingContinuation { continuation in
            MLModel.load(contentsOf: url) { result in
                switch result {
                case .success(let model):
                    continuation.resume(returning: model)
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
        }
    }
}
```

### 3. Privacy-First ML

```swift
// All processing on-device
actor PrivateAnalyzer {
    private let model: MLModel
    
    func analyze(sensitiveData: Data) async throws -> Result {
        // Never leaves device
        let input = try prepareInput(sensitiveData)
        let output = try await model.prediction(from: input)
        return parseResult(output)
    }
}
```

## 📚 Resources

- **Apple ML**: [developer.apple.com/machine-learning](https://developer.apple.com/machine-learning)
- **Core ML Tools**: [coremltools.readme.io](https://coremltools.readme.io)
- **WWDC 2025**: "Foundation Models on Apple Silicon"
- **Research**: [Efficient Transformers Survey](https://arxiv.org/abs/2009.06732)

## 🔗 Next Steps

- [Core ML Integration →](./coreml.md)
- [Create ML Workflows →](./createml.md)
- [Vision Framework →](./vision.md)

---

**Source**: Apple ML Documentation, Core ML Tools, WWDC 2025
