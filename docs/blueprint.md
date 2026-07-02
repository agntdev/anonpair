# Anonymous Chat Connector — Bot specification

**Archetype:** community

**Voice:** neutral and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that randomly matches anonymous users for one-on-one text, image, and sticker chats. Users remain anonymous with no visible Telegram handles. Sessions can be ended by either participant at any time, with moderation tools for reporting/blocking partners.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- General Telegram users seeking anonymous conversations
- Users interested in random one-on-one chats

## Success criteria

- Users can successfully find and chat with anonymous partners
- Moderation reports are delivered to admin chat within 1 minute
- Session state is preserved and properly ended when either user disconnects

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and onboarding flow
- **Find partner** (button, actor: user, callback: match:find) — Initiates the matching process based on language and non-blocked users
- **/find** (command, actor: user, command: /find) — Alternative command to initiate matching process
- **End chat** (button, actor: user, callback: session:end) — Ends current session for both users
- **Report** (button, actor: user, callback: report:start) — Opens report submission interface during active session
- **Block partner** (button, actor: user, callback: block:partner) — Blocks current partner and ends session
- **Next** (button, actor: user, callback: match:next) — Ends current session and immediately starts new matching process

## Flows

### Onboarding
_Trigger:_ /start

1. Display anonymity rules and terms
2. Request language selection
3. Request optional interests
4. Generate anonymous ID
5. Show main menu

_Data touched:_ user_profile

### Matching
_Trigger:_ match:find or /find

1. Add user to waiting pool
2. Filter by language and non-blocked users
3. Find random partner
4. Create session
5. Notify both users of match

_Data touched:_ match_session, user_profile

### Session Management
_Trigger:_ session active

1. Relay messages between partners
2. Handle 'End chat' command
3. Handle 'Report' command
4. Handle 'Block partner' command
5. Handle 'Next' command

_Data touched:_ match_session, block_list, reports

### Reporting
_Trigger:_ report:start

1. Show report options
2. Collect reason and optional text
3. Save report with session context
4. Forward to admin chat
5. End session

_Data touched:_ reports, match_session

### Blocking
_Trigger:_ block:partner

1. Add partner to user's block list
2. End current session
3. Notify user of block

_Data touched:_ block_list, match_session

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — Anonymous user data for matching and preferences
  - fields: anonymous_id, language, interests, block_list, opt_in_flags
- **match_session** _(retention: persistent)_ — Active and past one-on-one chat sessions
  - fields: session_id, user_a, user_b, status, start_time
- **reports** _(retention: persistent)_ — Moderation reports from users about partners
  - fields: report_id, reporter, reported_user, reason, context, timestamp
- **block_list** _(retention: persistent)_ — User-to-user blocking relationships
  - fields: user_a, user_b

## Integrations

- **Telegram** (required) — Bot API messaging and interface
- **Admin Chat** (required) — Moderation report notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin chat ID for reports
- Set message retention period for moderation
- Adjust matching algorithm weights for interests

## Notifications

- In-chat notification when match is found
- Admin chat notification when new report is submitted

## Permissions & privacy

- User data remains anonymous to partners
- Message content stored only for moderation purposes with configurable retention
- No access to Telegram user IDs or handles

## Edge cases

- User attempts to match while already in a session
- User tries to report/block during non-session state
- Both users end session simultaneously
- User selects same language but no available partners

## Required tests

- Verify anonymous ID generation and persistence
- Test session termination from both user perspectives
- Validate report forwarding to admin chat with context
- Confirm blocking prevents future matches
- Ensure message relay works for text/images/stickers

## Assumptions

- Users understand and accept anonymity rules
- Moderation reports will be reviewed by admin team
- Matching pool will have sufficient users for reasonable wait times
