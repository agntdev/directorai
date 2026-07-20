# DirectorAI — Bot specification

**Archetype:** content

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for directors/production specialists to convert scene-by-scene scripts into high-quality 4K H.265 videos (3-10 minutes). Supports both full-script submission and incremental scene addition with visual style confirmation, processing status updates, and iteration capabilities for keyframe previews and scene revisions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- directors
- production specialists

## Success criteria

- Generates 4K H.265 videos from text scripts
- Handles 20-60 second scenes with accurate visual style interpretation
- Delivers keyframe previews during processing
- Supports scene revisions and full project regeneration
- Manages file delivery within Telegram limits or via cloud links

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with project creation options
- **New project** (button, actor: user, callback: project:new) — Initiate new video project with default settings
  - inputs: project title, resolution/format
  - outputs: scene template form
- **Add scene** (button, actor: user, callback: scene:add) — Manually add individual scene to current project
  - inputs: scene text with visual cues
  - outputs: parsed scene confirmation
- **/submit_script** (command, actor: user, command: /submit_script) — Submit full script text for automated parsing

## Flows

### project_setup
_Trigger:_ /start or 'New project' button

1. Display welcome message
2. Request project title
3. Confirm default resolution/format
4. Begin scene collection

_Data touched:_ User, VideoJob

### scene_submission
_Trigger:_ scene:add or /submit_script

1. Parse scene text with AI
2. Confirm duration (default 30s)
3. Validate visual style elements
4. Store SceneBlock with metadata

_Data touched:_ SceneBlock, VideoJob

### processing_flow
_Trigger:_ project:finalize

1. Show processing start notification
2. Generate keyframe previews per scene
3. Send preview notifications
4. Assemble final video
5. Handle file delivery via Telegram or cloud link

_Data touched:_ VideoJob, GeneratedArtifact

### iteration_flow
_Trigger:_ preview:edit_request

1. Request specific scene revisions
2. Update SceneBlock content
3. Generate revised video segment
4. Send updated preview

_Data touched:_ SceneBlock, GeneratedArtifact

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with project history and preferences
  - fields: telegram_id, project_history, payment_status
- **SceneBlock** _(retention: persistent)_ — Parsed scene with visual and narrative metadata
  - fields: title, action_description, duration, camera_instructions, dialogue, music_style, visual_cues
- **VideoJob** _(retention: persistent)_ — Collection of scenes with processing metadata
  - fields: project_title, resolution, format, scene_sequence, status, total_duration
- **GeneratedArtifact** _(retention: persistent)_ — Video output files and previews
  - fields: artifact_type, file_url, scene_reference, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging and file delivery
- **Cloud Storage** (optional) — Large file hosting when Telegram limits exceeded
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete projects
- Edit scene content and visual style
- Set retry limits per scene
- Configure payment model (trial vs paid)

## Notifications

- Processing start confirmation
- Per-scene keyframe preview
- Final video delivery (file or link)
- Error notifications for invalid inputs

## Permissions & privacy

- Secure storage of user projects and scene data
- Optional payment information handling
- User consent for cloud storage usage

## Edge cases

- Handling video files exceeding Telegram's 2GB limit
- Invalid scene format in submitted scripts
- Payment failures during processing
- Scene retry limits exceeded

## Required tests

- End-to-end video generation from full script submission
- Preview delivery during multi-scene processing
- File delivery fallback to cloud storage
- Scene revision workflow with updated previews

## Assumptions

- Default resolution/format is 3840×2160 MP4 H.265
- Scene duration defaults to 30s when unspecified
- Max total video length enforced at 10 minutes
- Preview cadence is one keyframe per scene
