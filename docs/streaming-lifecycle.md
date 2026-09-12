[← Case study](../README.md) · [Run the evidence](verification.md)

# Streaming: the network and the UI finish at different times

**The final network event must not force pending text onto the screen.** Completion depends on both transport state and presentation state.

## The Level Up incident

AI chat text was arriving through SSE, but responses could still appear all at once. The final event flushed text that was queued for paced reveal.

I changed completion to wait until the reveal queue drained. A regression test delivers a burst, then releases the final event while some text is still hidden. It checks that streaming remains active and the send operation has not completed, then checks the final message after reveal finishes. A renderer test separately advances simulated time to check the completion callback.

The useful distinction is between **receiving the complete response** and **finishing its presentation**.

## Inspect the smallest version

The public [SSE decoder and reveal buffer][stream] separate two jobs:

- `decodeSse` carries UTF-8 and line boundaries across byte chunks, and joins `data:` lines into frames.
- `RevealBuffer` queues text for display. A done marker records transport completion; ticks drain the queue. Cancellation clears pending text and prevents late events from changing the visible prefix.

This trace matches the public test fixture, not a captured user conversation:

| Event | Visible text | Pending text | Complete? |
| :--- | :--- | :--- | :--- |
| Receive delta `abc` | Empty | `abc` | No |
| Receive `[DONE]` | Empty | `abc` | No |
| First reveal tick | `a` | `bc` | No |
| Second reveal tick | `ab` | `c` | No |
| Third reveal tick | `abc` | Empty | Yes |

The terminal states are distinct. Cancelled or malformed output does not become a successfully completed response. An EOF without the expected done marker is reported as interrupted.

## Tests worth opening

[The SSE tests][stream-tests] exercise:

- Completion arriving before the reveal finishes, with assertions after individual ticks.
- Cancellation preserving the visible prefix and ignoring later deltas.
- A multibyte character split across single-byte chunks and multiline CRLF frames.
- Invalid UTF-8, incomplete frames, malformed JSON, and EOF without a done marker.
- Cancelling the decoder subscription propagating cancellation to its byte source.

The critical test is `done before reveal finishes waits for incremental drain`: it checks intermediate state, so a final-text assertion alone cannot hide a regression.

## Tradeoffs and limits

Paced reveal deliberately keeps the UI active after the network finishes. That behavior needs cancellation and disposal handling, and intermediate-state tests.

The demo is a data-only SSE subset with local fixtures. It has no HTTP client, reconnect support, backpressure, or buffer limit. Graphemes are split within each delta; a combining sequence split between deltas is not coalesced. Those are explicit follow-up requirements before reusing it as a general streaming client.

**The lesson:** define what completion means for each layer, and test the period between those layers finishing.

[stream]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/lib/streaming/sse.dart
[stream-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/sse_test.dart
