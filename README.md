# Dynamic Relationships 0.2.1

Experimental UI extension for SillyTavern. Target baseline: SillyTavern 1.19.0.

## What is included

- Per-chat relationship state stored in `chatMetadata` (migrates the 0.1.0 metadata key).
- Automatic background analysis after `MESSAGE_RECEIVED`, using the currently selected model through `generateRaw()`.
- NPC discovery from names in the analyzed transcript, including group-chat message speakers.
- Directed/asymmetric relationship edges between user, main character, and NPCs.
- Metrics: trust, affection, respect, attachment, attraction (non-sexual desire for interpersonal closeness), fear, jealousy, irritation, admiration.
- Confidence threshold, bounded deltas, evidence/reason logging, and duplicate exchange protection.
- Relationship status labels derived from scores.
- Undo last applied relationship change.
- Managed lorebook projection saved alongside chat metadata.
- `{{dr_relationships}}` macro to insert the current projection into a prompt field that supports macros.
- Export of a World Info-compatible JSON entry for manual import.
- Read-only participant registry and event journal UI.

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

The extension keeps the canonical relationship data in the chat's metadata and maintains a managed lorebook projection there. It does not overwrite or edit existing user World Info books.

Two ways to use the projection:

1. Add `{{dr_relationships}}` to a prompt field that supports SillyTavern macros. It resolves to the latest managed relationship text for the currently open chat.
2. In the extension panel, click **Экспорт Lorebook**. Import the generated `Dynamic-Relationships-Lorebook.json` as a World Info book/entry using SillyTavern's World Info import UI.

The JSON export is a snapshot. Export again after subsequent relationship changes if you want a separately imported World Info book to reflect the latest state.

## Important limitations in 0.2.1

- The extension does not silently edit existing World Info books. It uses a chat-scoped projection, a prompt macro, and an explicit World Info JSON export. Direct programmatic World Info mutation is intentionally deferred until it can be verified against the exact 1.19.0 runtime API.
- `MESSAGE_EDITED`, `MESSAGE_DELETED`, and `MESSAGE_SWIPED` mark the chat as needing reconciliation; this build does not automatically reverse and replay the full journal after a branch/edit. Use Undo for the most recent individual change, or restore a chat backup if a large correction is needed.
- The model may misidentify entities or propose poor deltas. The confidence threshold and maximum delta limit reduce, but do not eliminate, this risk.
- Each automatic analysis is an additional model request and can add latency and API cost.
- This archive has been syntax-checked and package-checked, but not executed inside a live SillyTavern 1.19.0 instance. Runtime testing is still required.

## Data and privacy

All relationship state is stored in the current chat's metadata. The extension sends the selected conversation excerpt to the currently configured SillyTavern model for analysis, just as a normal generation request does. Do not use it with chats whose contents you do not want sent to that provider.

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

The manifest declares `minimum_client_version: 1.19.0`. The extension uses documented UI extension facilities including `getContext()`, `chatMetadata`, `saveMetadata()`, `eventSource`, `MESSAGE_RECEIVED`, `CHAT_CHANGED`, `generateRaw()`, and the macro registry where available.

Documentation:
- https://docs.sillytavern.app/for-contributors/writing-extensions/
- https://docs.sillytavern.app/usage/core-concepts/worldinfo/
