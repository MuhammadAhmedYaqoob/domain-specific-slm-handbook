# iOS offline SLM integration

## 1. Runtime choices

Core ML offers Apple packaging and CPU/GPU/ANE eligibility but may leave tokenization and generation to the app. MLX
Swift LM provides an Apple CPU/GPU generation stack; it is not an ANE runtime. llama.cpp offers GGUF with CPU/Metal.
ExecuTorch uses backend-specialized programs. Select by exact model and deployment target, then prove placement with
Instruments rather than assuming `.all` means ANE.

## 2. Worked iPhone example

For a modern iPhone cohort, begin with the same 1B low-bit model and 2,048-token cap. The illustrative 96 MiB KV
estimate is only one allocation. Use Instruments to measure resident/expanded weights, VM footprint, prefill workspace,
Metal/Core ML programs, and UI. A candidate passes only if a signed Release build remains below the registered memory
gate and maintains acceptable early and late throughput without reaching critical thermal state.

## 3. Step-by-step build and integration

1. On macOS, create a SwiftUI project and set the oldest supported iOS deployment target.
2. Pin one runtime package revision and all required tokenizer/kernel products.
3. Convert a backend-specific artifact; preserve model, tokenizer, converter, and runtime hashes.
4. Compare source and target token IDs, prompt bytes, greedy outputs/logits, stops, and cache reset.
5. Bundle a small immutable artifact or install a downloaded model in `Library/Application Support`.
6. Verify the signed package before the engine parses it.
7. Own engine and KV state in one actor; keep load, hashing, prefill, and decode off the main actor.
8. Publish fragments to an `@MainActor` ViewModel and handle cancellation.
9. Observe `scenePhase`; cancel or pause by tested policy before suspension.
10. Archive and sign a Release build; test the oldest and representative physical devices.

## 4. Swift integration

The protocol isolates UI from runtime-specific APIs:

```swift
import Foundation

protocol LocalGenerating: Sendable {
    func load(modelURL: URL, contextTokens: Int) async throws
    func stream(prompt: String, maxNewTokens: Int)
        -> AsyncThrowingStream<String, Error>
    func cancel() async
    func reset() async throws
}

actor LocalEngine: LocalGenerating {
    private var session: RuntimeSession?

    func load(modelURL: URL, contextTokens: Int) async throws {
        session = try RuntimeSession(modelURL: modelURL, contextTokens: contextTokens)
    }

    func stream(prompt: String, maxNewTokens: Int)
        -> AsyncThrowingStream<String, Error> {
        guard let session else {
            return AsyncThrowingStream { $0.finish(throwing: EngineError.notLoaded) }
        }
        return session.stream(prompt: prompt, maximumTokens: maxNewTokens)
    }

    func cancel() async { session?.cancel() }
    func reset() async throws { try session?.reset() }
}
```

`RuntimeSession` is a small adapter implemented against the pinned Core ML, MLX, llama.cpp, or ExecuTorch release.
This separation makes template, lifecycle, and UI tests independent from backend churn. Its stream must buffer
incomplete UTF-8 and stop callbacks before reset returns.

```swift
@MainActor
final class ChatViewModel: ObservableObject {
    @Published private(set) var answer = ""
    @Published private(set) var isGenerating = false
    private let engine: LocalGenerating
    private var task: Task<Void, Never>?

    init(engine: LocalGenerating) { self.engine = engine }

    func send(renderedPrompt: String) {
        task?.cancel()
        answer = ""
        isGenerating = true
        task = Task {
            do {
                for try await fragment in engine.stream(
                    prompt: renderedPrompt, maxNewTokens: 256
                ) {
                    try Task.checkCancellation()
                    answer += fragment
                }
            } catch { await engine.cancel() }
            isGenerating = false
        }
    }

    func backgrounded() {
        task?.cancel()
        Task { await engine.cancel() }
    }
}
```

Expected behavior is responsive streaming on the main actor while all model work remains serialized elsewhere.
Implement the exact training chat template; never substitute a family-wide default without byte-for-byte tests.

## 5. Build, storage, and measurement

```bash
xcodebuild -project OfflineSpecialist.xcodeproj \
  -scheme OfflineSpecialist -configuration Release \
  -destination 'generic/platform=iOS' CODE_SIGNING_ALLOWED=NO build
```

This probes compilation and linking only. Physical installation requires a team, unique bundle identifier, provisioning,
archive, and normal signing. Store durable downloaded models in Application Support, staging in the temporary directory,
and only reproducible caches in Caches. Choose file protection based on locked-device requirements.

Use Allocations and VM Tracker for memory, Time Profiler and `os_signpost` for stages, Core ML Instruments for compute
units, and Metal tools for GPU work. Test battery operation separately from charging.

## 6. Common failures and validation

- Package/link errors: align package products, kernels, registration, and linker flags.
- Bundled path is missing: check target membership, case, and Copy Bundle Resources.
- Memory termination: inspect duplicate/expanded weights, KV, context, and prefill temporaries.
- Delayed stop: serialize cancellation and require callback quiescence before reset.
- Simulator mismatch: use it for UI tests only; it is not device memory, Metal, or performance evidence.

Validate signature and corruption failures, tokenizer/template/logit/cache goldens, cold/warm load, multi-turn session
isolation, context limit, lock/background/termination, memory pressure, placement, network-denied operation, low-space
update, rollback, and sustained signed-Release behavior.

## Sources

- [Core ML](https://developer.apple.com/documentation/coreml)
- [Core ML Tools optimization](https://apple.github.io/coremltools/docs-guides/source/opt-overview.html)
- [MLX Swift LM](https://github.com/ml-explore/mlx-swift-lm)
- [ExecuTorch on iOS](https://docs.pytorch.org/executorch/stable/using-executorch-ios.html)
- [Apple thermal state](https://developer.apple.com/documentation/foundation/processinfo/thermalstate-swift.property)
