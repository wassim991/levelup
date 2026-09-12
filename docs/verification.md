[← Case study](../README.md)

# Run the evidence

The case study describes Level Up engineering work. The repositories below are standalone public examples; their tests verify those examples, not the private app or its deployed backend.

## Verified snapshot · September 12, 2026

| Repository | Revision | Local result | CI |
| :--- | :--- | :--- | :--- |
| Structured-output pipeline | [`3eccf72`](https://github.com/wassim991/llm-structured-output-pipeline/tree/3eccf725b333367cd9539aa980ade9d8d294c7b9) | 26 tests passed; format and lint passed; fixture demo ran | [Successful run](https://github.com/wassim991/llm-structured-output-pipeline/actions/runs/34646595714) |
| Flutter offline sync demo | [`30b143d`](https://github.com/wassim991/flutter-offline-sync-demo/tree/30b143d39f8db679cb605e176ed87c17a4dd88bd) | 17 tests passed | [Successful run](https://github.com/wassim991/flutter-offline-sync-demo/actions/runs/34646982143) |

Source and test links in this case study point to these exact revisions. Test counts describe this snapshot, not a claim about future commits. Local verification used macOS, Deno 2.6.4, and Flutter 3.41.1 / Dart 3.11.0.

## Structured output

With Deno 2.6.4 installed:

```sh
git clone https://github.com/wassim991/llm-structured-output-pipeline.git
cd llm-structured-output-pipeline
git checkout 3eccf725b333367cd9539aa980ade9d8d294c7b9
deno task check
deno task demo
```

The first run downloads pinned dependencies. The demo prints three fixture outcomes: valid first response, malformed response repaired, and repair exhausted. No model credentials are needed.

Start with [`run`][pipeline] and the [failure-path tests][pipeline-tests]. The injected provider is scripted; no live model evaluation is included.

## Offline sync and SSE

With Flutter 3.41.1 installed:

```sh
git clone https://github.com/wassim991/flutter-offline-sync-demo.git
cd flutter-offline-sync-demo
git checkout 30b143d39f8db679cb605e176ed87c17a4dd88bd
flutter pub get --enforce-lockfile
flutter test
```

To inspect the UI on macOS with Xcode configured:

```sh
flutter run -d macos
```

The sync screen exposes offline writes, a lost acknowledgement, retries, and duplicate echoes. The SSE lab exposes normal, interrupted, and malformed fixtures. Use disposable notes: the local database persists; the simulated server does not.

Start with [transaction tests][store-tests], [retry tests][sync-tests], and [stream lifecycle tests][stream-tests].

## What these checks establish

| Evidence | Scope |
| :--- | :--- |
| App Store link | Public product listing |
| Local example tests and linked CI | Behavior of the pinned public implementations under the stated scenarios |
| Disk restart test | Local SQLite state and pending operation identity survive reopening |
| Scripted model and server fixtures | Reproducible control-flow failures without external services |

These runs do not establish model quality, live-service reliability, multi-device correctness, or current production release readiness. The original Level Up incidents and live preflight checks are engineering history; they are not recreated by the public test suites.

[pipeline]: https://github.com/wassim991/llm-structured-output-pipeline/blob/3eccf725b333367cd9539aa980ade9d8d294c7b9/src/pipeline.ts
[pipeline-tests]: https://github.com/wassim991/llm-structured-output-pipeline/blob/3eccf725b333367cd9539aa980ade9d8d294c7b9/tests/pipeline_test.ts
[stream-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/sse_test.dart
[store-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/store_test.dart
[sync-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/sync_test.dart
