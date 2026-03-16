# Joebot Ecosystem — Documentation
🦖 Architecture docs for the Joebot studio ecosystem.

| Document | Description |
|---|---|
| Nexus_Architecture.md | Grand server spec |
| JBT_Format_Spec.md | Shared file format |
| GlitchBoard_Spec.md | DAW-style show control build specification |
| NexusControl_BuildGuide.md | Device administration app build guide |
| LyricApp_BuildGuide.md | Lyric authoring and TextWall sync build guide |
| TextWall_BuildGuide.md | Text and lyric display app build specification |
| DirtyMixerApp_BuildGuide.md | Dirty mixer controller app |
| JoebotSDK_Guide.md | Shared Swift toolkit |
| Observatory_BuildGuide.md | Dashboard and launcher app |
| SessionReplay_Spec.md | Session replay feature spec |
| Extron_SIS_Reference.md | Extron device command reference |
| ThreeGreenDots_BuildPrompt.md | Three app proof of concept prompt |
| Nexus_ClaudeCode_Prompt_v1.md | Nexus build session prompt |

## March 2026 Update Notes

- `Nexus_Architecture.md` v1.2 -> v1.3: event logger, session packaging, operating modes, layout query, `.jbt` mandate, port fixes.
- `JBT_Format_Spec.md` v1.0 -> v1.1: added `textwall_layout`, `lyric_timeline` v2.0, `daw_setlist`, `nexus_device_registry`, `nexus_eir_library`, `midi_mapping`, `app_prefs`, `capabilities_cache`.
- `JoebotSDK_Guide.md` v1.0 -> v1.1: updated theme colors (Amber fix), new Neo Cyberpunk + Phosphor Green themes, `OperatingModeToggle`, `TextWallPreview`, `CellLayoutPicker`, port fix.
- `GlitchBoard_Spec.md` v1.2 -> v1.3: TextWall lane, cue editors with previews, SIS hover tooltips, snake keyframe editor, freeform cell picker, duplicate stamp tool, lyric import, "Welcome to the Machine" performance map.
- `NexusControl_BuildGuide.md` v1.0 -> v1.1: MIDI tab (APC Mini virtual layout), RGB button color picker, operating mode badges, input/output/preset naming, VSC baud config, Apps tab.
- `LyricApp_BuildGuide.md` v1.0 -> v2.0: full TextWall config per cue, Phase 0 deadline build, mini preview, `.jbt` v2.0.
- `TextWall_BuildGuide.md` v1.0: new document covering all modes, scatter, star layout, and performance map.
