0) Legend & Conventions

PK: Primary Key

FK →: Foreign Key reference

IX: Indexed field

Enum: Enumerated values

JSON: JSON-typed field (schematized where relevant)

All timestamps in UTC ISO 8601 (YYYY-MM-DDTHH:MM:SSZ)
1) Core Identity & Access
1.1 User

user_id (PK)

name (e.g., “Rocco”)

email (IX, unique)

role (enum: creator, admin)

created_at

last_login_at

preferences_json (JSON; ui_theme, notif_prefs, default_platforms[])

status (enum: active, disabled)

Relationships

Has many Agent

Has many SocialAccount

Owns Project, Campaign, AI_Twin, Script, ContentCalendar, etc.
1.2 Agent

agent_id (PK)

user_id (FK → User)

name (default: “AI Twin Content Agent”)

state (enum: idle, planning, creating, publishing, analyzing, adapting, error)

capabilities_json (JSON; e.g., {planner:true, media:true, publisher:true, analytics:true, learning:true})

created_at

updated_at

Relationships

Has one Agent_Memory

Uses many IntegrationKey

Produces RenderJob, VideoReel, Post, WeeklyReport, StrategyUpdate
1.3 IntegrationKey

integration_id (PK)

user_id (FK → User)

platform (enum: instagram, facebook, threads, linkedin, tiktok, youtube_shorts, fanbase, spill)

client_app_name

access_token (encrypted)

refresh_token (encrypted, nullable)

token_expires_at (nullable)

scopes (string[])

created_at

updated_at

status (enum: connected, revoked, expired)

Relationships

Powers SocialAccount

1.4 SocialAccount

account_id (PK)

user_id (FK → User)

integration_id (FK → IntegrationKey)

platform (enum as above)

handle (IX)

account_external_id (IX)

is_connected (boolean)

last_synced_at

created_at

updated_at

Relationships

Has many Post

Used by PublisherTask
2) Planning & Strategy
2.1 Project

project_id (PK)

user_id (FK → User)

name

description

objectives_json (JSON; KPIs, audience segment, tone)

start_date

end_date (nullable)

status (enum: active, paused, completed)

created_at

updated_at

2.2 Campaign

campaign_id (PK)

project_id (FK → Project)

name

description

target_platforms (string[]) // e.g., ["tiktok","instagram","linkedin"]

budget_json (JSON; optional for paid boosts)

start_date

end_date

status (enum: planned, running, completed, cancelled)

created_at

updated_at

2.3 ContentCalendar

calendar_id (PK)

user_id (FK → User)

project_id (FK → Project, nullable)

period_start

period_end

version (int)

strategy_summary (text)

created_at

updated_at

Relationships

Has many CalendarItem

2.4 CalendarItem

calendar_item_id (PK)

calendar_id (FK → ContentCalendar)

campaign_id (FK → Campaign, nullable)

planned_date

planned_platforms (string[])

content_pillar (enum: education, inspiration, announcement, promotion, trend_reaction)

ai_twin_id (FK → AI_Twin, nullable)

script_id (FK → Script, nullable)

status (enum: planned, in_progress, render_ready, published, skipped)

created_at

updated_at

2.5 TrendSignal

trend_id (PK)

source (enum: google_trends, platform_trending, hashtag_feed, news)

topic (IX)

platform_hint (nullable; enum as above)

score (float) // significance

collected_at

metadata_json (JSON; region, related_tags, velocity)

3) Script & Creative Assets
3.1 Script

script_id (PK)

user_id (FK → User)

campaign_id (FK → Campaign, nullable)

title

body_text (text) // ≤150 words target; agent can shorten

tone (enum: educational, trending, thought_leadership, inspirational)

language (ISO code, default: en)

tags (string[]) // e.g., ["web3","ai","founders"]

platform_targets (string[]) // e.g., ["tiktok","instagram"]

created_at

updated_at

status (enum: draft, approved, rejected)

edit_history_json (JSON array [{at, editor, delta}])

3.2 AI_Twin

ai_twin_id (PK)

user_id (FK → User)

name (e.g., “Space Twin”, “Goddess Twin”, “Hyperreal Twin”)

style_preset (enum: space, goddess, hyperreal, corporate, custom)

voice_profile_id (FK → VoiceProfile, nullable)

config_json (JSON; facial map, lighting, backgrounds, attire)

avatar_assets_folder (URI)

created_at

updated_at

status (enum: active, archived)

3.3 VoiceProfile

voice_profile_id (PK)

user_id (FK → User)

provider (enum: elevenlabs, resemble, other)

name

voice_id_external (string)

settings_json (JSON; pitch, speed, timbre)

created_at

updated_at

3.4 MediaAsset

asset_id (PK)

user_id (FK → User)

type (enum: raw_video, raw_photo, audio, overlay, caption_file, thumbnail, brand_pack)

uri (string)

duration_ms (nullable)

resolution (e.g., “1080x1920”)

metadata_json (JSON; EXIF, color_lut, source_app)

created_at

updated_at

Relationships

Used by RenderJob and VideoReel

4) Rendering & Video Output
4.1 RenderJob

render_id (PK)

user_id (FK → User)

ai_twin_id (FK → AI_Twin)

script_id (FK → Script)

input_assets (JSON; asset_ids[])

tts_voice_profile_id (FK → VoiceProfile)

target_aspect (enum: 9_16, 1_1, 4_5)

target_duration_sec (int, ≤ 30)

status (enum: queued, processing, succeeded, failed, cancelled)

progress_pct (float)

output_asset_id (FK → MediaAsset, nullable)

error_json (JSON, nullable)

created_at

updated_at

4.2 VideoReel

reel_id (PK)

user_id (FK → User)

render_id (FK → RenderJob)

title

caption_text (text)

hashtags (string[])

length_sec (int)

aspect_ratio (enum: 9_16, 1_1, 4_5)

quality_score (float, nullable) // internal heuristic

storage_uri (string) // final mp4

thumbnail_asset_id (FK → MediaAsset, nullable)

created_at

updated_at

approval_status (enum: auto_approved, creator_approved, rejected)

Relationships

Feeds Post

5) Publishing & Distribution
5.1 Post

post_id (PK)

user_id (FK → User)

reel_id (FK → VideoReel)

account_id (FK → SocialAccount)

platform (enum as above)

caption (text)

scheduled_time

publish_status (enum: draft, scheduled, publishing, published, failed)

published_at (nullable)

platform_post_id (string, IX, nullable)

error_json (JSON, nullable)

created_at

updated_at

5.2 PublisherTask

task_id (PK)

post_id (FK → Post)

attempt (int)

status (enum: queued, in_progress, succeeded, failed, cancelled)

worker_node (string, nullable)

response_json (JSON; raw API response)

error_json (JSON, nullable)

created_at

updated_at

6) Analytics & Learning
6.1 AnalyticsSnapshot

snapshot_id (PK)

post_id (FK → Post)

platform (enum as above)

captured_at (IX)

metrics_json (JSON; {views, impressions, likes, comments, shares, saves, watch_time_sec, ctr, followers_delta})

notes (nullable)

6.2 WeeklyReport

report_id (PK)

user_id (FK → User)

week_start (IX)

week_end

aggregate_metrics_json (JSON; totals + per-platform breakdown)

top_posts (JSON; [post_id, score, title, link])

insights (text)

pdf_uri (nullable)

created_at

6.3 StrategyUpdate

strategy_id (PK)

user_id (FK → User)

source_report_id (FK → WeeklyReport)

adjustments_json (JSON; {topics_to_increase[], topics_to_reduce[], platform_timing_updates, tone_adjustments, twin_rotation})

effective_start_date

created_at

6.4 Agent_Memory

memory_id (PK)

agent_id (FK → Agent)

vector_db_uri (string)

index_stats_json (JSON; dims, size, last_compaction_at)

retention_policy_json (JSON)

created_at

updated_at

6.5 Feedback

feedback_id (PK)

user_id (FK → User)

subject_type (enum: script, reel, post, agent)

subject_id (string)

rating (int 1–5)

comment (text, nullable)

created_at

7) Interaction Layer (Mobile & Web)
7.1 AppSession

session_id (PK)

user_id (FK → User)

device_type (enum: ios, android, web)

device_info_json (JSON; model, os, app_version)

ip_address

started_at

ended_at (nullable)

status (enum: active, closed)

7.2 Notification

notification_id (PK)

user_id (FK → User)

type (enum: render_complete, publish_success, publish_failed, weekly_report_ready, strategy_update)

payload_json (JSON; deeplink target)

is_read (boolean)

created_at

8) Orchestration & Audit
8.1 AgentRun

run_id (PK)

agent_id (FK → Agent)

trigger_type (enum: schedule_weekly, manual, trend_spike, retry, system)

started_at

finished_at (nullable)

result (enum: success, partial_success, error)

log_uri (string, nullable)

summary (text)

8.2 EventLog

event_id (PK)

user_id (FK → User, nullable)

agent_id (FK → Agent, nullable)

event_type (enum: plan_created, script_generated, render_started, render_succeeded, publish_scheduled, publish_succeeded, analytics_collected, strategy_adjusted, error)

subject_type (enum: calendar, script, render, reel, post, report, strategy, system)

subject_id (string, nullable)

message (text)

created_at (IX)

meta_json (JSON)

8.3 ErrorLog

error_id (PK)

scope (enum: planner, media, publisher, analytics, learning, api_integration, ui)

context_json (JSON)

message (text)

stack_trace (text, nullable)

occurred_at

severity (enum: low, medium, high, critical)

resolved_at (nullable)

resolution_notes (nullable)

9) Governance & Security
9.1 Role

role_id (PK)

name (enum: creator, admin)

permissions_json (JSON; feature flags)

9.2 UserRole

user_role_id (PK)

user_id (FK → User)

role_id (FK → Role)

granted_at

9.3 ConsentRecord

consent_id (PK)

user_id (FK → User)

consent_type (enum: likeness_modeling, voice_cloning, data_processing, api_publishing)

granted (boolean)

granted_at

details (text)

10) Key Enums

Platforms: instagram, facebook, threads, linkedin, tiktok, youtube_shorts, fanbase, spill

Content Pillars: education, inspiration, announcement, promotion, trend_reaction

Agent State: idle, planning, creating, publishing, analyzing, adapting, error

Aspect Ratios: 9_16, 1_1, 4_5
11) High-Level Relationships (ASCII ER Sketch)
User ─┬─< Agent ──┬─ 1:1 Agent_Memory
      ├─< IntegrationKey ──< SocialAccount ──< Post ──< AnalyticsSnapshot
      ├─< Project ──< Campaign
      ├─< ContentCalendar ──< CalendarItem
      ├─< AI_Twin ──< RenderJob ──< VideoReel ──< Post
      ├─< VoiceProfile
      ├─< MediaAsset
      ├─< Script
      ├─< WeeklyReport ──< StrategyUpdate
      ├─< Feedback
      ├─< Notification
      └─< AppSession

Agent ─< AgentRun ─< EventLog
12) Suggested Indexing & Performance Notes

Hot paths:

Post(platform_post_id) (IX) for analytics joins

AnalyticsSnapshot(post_id, captured_at) composite IX

RenderJob(status, created_at) IX for job queue polling

CalendarItem(planned_date) IX for weekly views

VideoReel(approval_status, created_at) IX for review queues

Large JSON fields (metrics, configs) should use JSONB (Postgres) with GIN indexes where filtering is needed.

13) Example Payloads (abbreviated)
13.1 Generated Strategy Update (adjustments_json)
{
  "topics_to_increase": ["Web3 security", "Creator AI tools"],
  "topics_to_reduce": ["Crypto price commentary"],
  "platform_timing_updates": {
    "tiktok": ["Mon 11:30", "Thu 19:00"],
    "instagram": ["Wed 12:00"]
  },
  "tone_adjustments": { "linkedin": "thought_leadership" },
  "twin_rotation": ["Hyperreal Twin", "Goddess Twin"]
}
13.2 Analytics Snapshot (metrics_json)
{
  "views": 18245,
  "impressions": 24500,
  "likes": 1340,
  "comments": 88,
  "shares": 210,
  "saves": 305,
  "watch_time_sec": 54210,
  "ctr": 0.043,
  "followers_delta": 120
}
14) Data Lifecycle & Retention

Raw assets: 90–180 days (configurable), archived to cold storage after publish.

Agent logs & events: 180–365 days (PII-scrubbed).

Vector memory: rolling compaction; maintain top-K relevance per topic/twin/persona.

Consent records: retained for account lifetime.

15) Privacy & Compliance Notes

Encrypt tokens, refresh tokens, and any biometric/likeness configuration.

Store only necessary identifiers from platforms.

Provide export/delete for user data (GDPR/CCPA support).

Maintain explicit consent for likeness/voice usage (ConsentRecord).
