# Dynamic Relationships 0.4.1

Experimental UI extension for SillyTavern. Target baseline: SillyTavern 1.19.0.

## What is included

- Per-chat relationship state stored in `chatMetadata` (migrates the 0.1.0 metadata key).
- Automatic background analysis after `MESSAGE_RECEIVED`, using the currently selected model through `generateRaw()`.
- NPC discovery from names in the analyzed transcript, including group-chat message speakers.
- Directed/asymmetric relationship edges between the user and story participants (main character tracking can be disabled), including NPC-to-NPC.
- Metrics: trust, affection, respect, attachment, attraction (non-sexual desire for interpersonal closeness), fear, jealousy, irritation, admiration.
- Confidence threshold, bounded deltas, evidence/reason logging, and duplicate exchange protection.
- Human-readable, multi-part relationship status descriptions (for example, “союзники / взаимное уважение”), alongside the numeric-derived internal status.
- Undo last applied relationship change.
- Managed lorebook projection saved alongside chat metadata.
- `{{dr_relationships}}` macro to insert the current projection into a prompt field that supports macros.
- Export of a World Info-compatible JSON entry for manual import.
- Read-only participant registry and event journal UI.
- Per-chat option to exclude `{{char}}` from relationship tracking (useful for neutral narrator bots).
- Per-chat selection of an existing World Info / Lorebook for persistence. The extension creates or updates only its own marked entry and preserves the other entries in that book.
- Automatic update of the selected Lorebook entry after each analysis, plus a manual sync button.
- Optional scene tracking for in-story date/time, weather, and location; can be enabled or disabled from the extension panel. When disabled, scene extraction and message footer output stop, while previously collected scene data remains stored. The extension adds a styled “ТЕКУЩИЙ СЕТТИНГ” footer to assistant messages and includes the latest scene state in the Lorebook projection.

### Attraction metric

`attraction` tracks a character’s non-sexual interpersonal pull toward another person: seeking contact or attention, wanting to spend time together, and wanting to reduce emotional distance. It is distinct from `affection` (positive liking) and `attachment` (enduring emotional bond). The analyzer must not infer or score sexual desire. Attraction alone is not evidence of manipulation; coercive or manipulative dynamics should be noted only when the transcript provides concrete evidence.

Existing chats are migrated automatically: older relationships receive a neutral starting value of 50 for the new metric.

## Install

1. Make a backup of your SillyTavern folder and test on a disposable chat first.
2. Extract `DynamicRelationships` from the ZIP.
3. Copy the folder into:
   `SillyTavern/data/<your-user-handle>/extensions/DynamicRelationships/`
4. Reload the SillyTavern page (or restart the server if required by your setup).
5. Open Extensions and enable **Dynamic Relationships**.
6. Open a chat and generate a character reply. The extension will issue an additional quiet/raw analysis request after the reply is recorded.

## Lorebook / World Info workflow

The extension keeps canonical relationship data in the chat's metadata and maintains a managed projection. In the extension panel, choose an existing World Info book under **Lorebook для сохранения**. The extension creates one dedicated entry marked in its `extensions` metadata and updates that entry after each analysis. Other entries in the selected book are preserved. You can also press **Сохранить в Lorebook** to sync manually. Choose **Только внутреннее хранилище чата** to disable writes to a World Info book.

The extension does not delete old managed entries from a book when you change the target book; if you switch destinations, the previous book retains its last saved snapshot.

Two additional ways to use the projection:

1. Add `{{dr_relationships}}` to a prompt field that supports SillyTavern macros. It resolves to the latest managed relationship text for the currently open chat.
2. In the extension panel, click **Экспорт Lorebook**. Import the generated `Dynamic-Relationships-Lorebook.json` as a World Info book/entry using SillyTavern's World Info import UI.

The JSON export is a snapshot. Export again after subsequent relationship changes if you want a separately imported World Info book to reflect the latest state.

## Scene tracking and message footer

The module is controlled by the **Отслеживать сеттинг: дата, время, погода и локация** checkbox in the extension panel. It is enabled by default. When disabled, the analyzer stops extracting scene fields, the footer is hidden (including on already rendered messages), and scene fields are omitted from the Lorebook projection. Previously collected scene values are retained in chat metadata and become available again if the module is re-enabled.


After each assistant reply, the same background analysis request also extracts the current in-story time/date, weather, and location when they are explicitly stated or unambiguously established in the recent transcript. Missing or uncertain fields are left blank; the extension must not invent exact times, weather, or locations. Known scene values carry forward until the story establishes a change. The footer is attached through SillyTavern’s message formatting hook and is presentation-only: it does not rewrite the stored assistant message text.

The scene information is generated from roleplay context, not from the device clock, geolocation, or a live weather service.

Relationship status descriptions are natural-language labels proposed by the model from the transcript (such as “наставник и ученица / тайная связь”). They are separate from the score-derived internal category. Treat them as narrative summaries, not objective psychological diagnoses.

## Important limitations in 0.4.1

- Lorebook saving uses SillyTavern 1.19.0 World Info endpoints (`/api/worldinfo/list`, `/get`, `/edit`). It updates only an entry carrying this extension's chat-specific marker. Test on a backup first; simultaneous edits to the same Lorebook from multiple browser sessions can still cause last-write-wins behavior.
- `MESSAGE_EDITED`, `MESSAGE_DELETED`, and `MESSAGE_SWIPED` mark the chat as needing reconciliation; this build does not automatically reverse and replay the full journal after a branch/edit. Use Undo for the most recent individual change, or restore a chat backup if a large correction is needed.
- The model may misidentify entities or propose poor deltas. The confidence threshold and maximum delta limit reduce, but do not eliminate, this risk.
- Each automatic analysis is an additional model request and can add latency and API cost.
- This archive has been syntax-checked and package-checked, but not executed inside a live SillyTavern 1.19.0 instance. Runtime testing is still required, especially the World Info save workflow.

## Data and privacy

All relationship state is stored in the current chat's metadata. The extension sends the selected conversation excerpt to the currently configured SillyTavern model for analysis, just as a normal generation request does. Do not use it with chats whose contents you do not want sent to that provider.

## Narrator exclusion

The **Не отслеживать {{char}} (нейтральный рассказчик)** checkbox is stored per chat. When enabled, the current main character name is omitted from entity discovery, proposed relationship edges, the visible relationship projection, and future analysis instructions. Previously recorded `{{char}}` relationship edges are retained in chat metadata but hidden from the panel and Lorebook projection while the option is enabled; turning it off makes them visible again. New edges involving `{{char}}` are ignored. The user's own entity and NPC-to-NPC relationships continue to be tracked.

## Settings

Defaults are conservative:
- Automatic analysis: on
- Transcript window: 12 messages
- Minimum confidence: 0.68
- Maximum metric delta per analyzed event: 8 points
- Maximum entities: 80
- Maximum journal entries: 600

Settings are stored in SillyTavern extension settings under `dynamic_relationships`.

## Compatibility

The manifest declares `minimum_client_version: 1.19.0`. The extension uses documented UI extension facilities including `getContext()`, `chatMetadata`, `saveMetadata()`, `eventSource`, `MESSAGE_RECEIVED`, `CHAT_CHANGED`, `generateRaw()`, the macro registry, and the message formatter hook where available.

Documentation:
- https://docs.sillytavern.app/for-contributors/writing-extensions/
- https://docs.sillytavern.app/usage/core-concepts/worldinfo/
