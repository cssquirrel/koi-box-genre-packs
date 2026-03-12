# Koi Box Genre Packs

Community-made genre packs for [Koi Box](https://github.com/cssquirrel/koi-box), a desktop radio that generates its own music using [ACE-Step 1.5](https://github.com/ace-step/ACE-Step-1.5).

A genre pack defines everything Koi Box needs to generate music for a category: the ACE-Step prompts (captions, lyrics templates, metadata), the name/artist/album generators, album cover art, and optional LLM prompts for dynamic lyrics and artist bios.

---

## Creating Your Own Genre Pack

This guide walks you through building a genre pack from scratch. We'll reference the included **K-Alt** pack (`packs/cssquirrel/k-alt/`) as a working example throughout.

### Prerequisites

Before you start, you should:

- Have [Koi Box](https://github.com/cssquirrel/koi-box) installed and working
- Have [ACE-Step 1.5](https://github.com/ace-step/ACE-Step-1.5) running (its API on port 8001)
- Read the [ACE-Step 1.5 Tutorial](https://github.com/ace-step/ACE-Step-1.5/blob/main/docs/en/Tutorial.md) to understand how captions, lyrics tags, BPM, key, and duration affect music generation
- Have album cover images ready (JPG format recommended)

---

### Step 1: Plan Your Pack

Before creating any files, decide on:

1. **Category concept** — What genre or sonic world does your pack cover? (e.g., "K-Alt" covers Korean alt-rock and nu-metal)
2. **Variants** — What distinct sub-styles exist within it? Each variant gets its own ACE-Step caption, lyrics template, BPM range, key, and mood. Aim for 2-6 variants that are clearly different from each other.
3. **Lyrics approach** — Will your tracks be instrumental (`[Instrumental]`), use static lyrics templates, or use an LLM to dynamically generate lyrics per song?
4. **Pack ID** — A short lowercase identifier (e.g., `kalt`, `lofi`, `synthwave`). This must be unique across all installed packs.

**K-Alt example:** The K-Alt pack has four variants covering a spectrum from emotional alt-rock (`lee-park`) to female rap-rock (`bite-back`), all sharing a Korean alt-rock / nu-metal identity. It uses LLM-generated Korean lyrics.

---

### Step 2: Set Up the Directory Structure

Genre packs live under `packs/<your-username>/<pack-name>/`. Create the following structure:

```
packs/
  <your-username>/
    <pack-name>/
      pack.manifest                          # Pack metadata (JSON)
      album_covers/
        <category_id>/
          cover-0001.jpg                     # Album cover images
          cover-0002.jpg
          ...
      config/
        category_info.yaml                   # Category display & engine settings
        genre_info.yaml                      # Variant definitions (captions, lyrics, etc.)
        generators/
          profiles/
            <profile-name>.yaml              # Name generation templates
          pools/
            <category_id>/
              adjectives.txt                 # Word pool files (one word per line)
              nouns.txt
              verbs.txt
              ...
          prompts/                           # (Optional) LLM system prompts
            <category>_lyrics.txt
            <category>_bios.txt
```

**K-Alt example structure:**

```
packs/cssquirrel/k-alt/
  pack.manifest
  album_covers/k_alt/
    k_alt-0001.jpg ... k_alt-0198.jpg
  config/
    category_info.yaml
    genre_info.yaml
    generators/
      profiles/kalt-rock.yaml
      pools/kalt/
        adjectives.txt, nouns.txt, verbs.txt, emotions.txt, suffixes.txt,
        artist_prefixes.txt, artist_suffixes.txt, artist_words.txt,
        artist_words2.txt, artist_solo.txt, artist_hangul.txt, artist_feat.txt,
        album_adjectives.txt, album_nouns.txt, album_articles.txt
      prompts/
        kalt_lyrics.txt
        kalt_bios.txt
```

---

### Step 3: Create the Pack Manifest

Create `pack.manifest` in your pack's root directory. This is a JSON file that identifies your pack:

```json
{
  "pack_format": 1,
  "id": "mypack",
  "name": "My Pack Display Name",
  "description": "A short description of what this pack sounds like.",
  "version": "1.0.0",
  "author": "your-username",
  "category_id": "mypack",
  "variants": ["variant-one", "variant-two", "variant-three"]
}
```

| Field | Description |
|-------|-------------|
| `pack_format` | Always `1` (current format version) |
| `id` | Unique pack identifier (lowercase, no spaces) |
| `name` | Human-readable display name |
| `description` | Brief description of the pack's sound/concept |
| `version` | Semantic version (e.g., `1.0.0`) |
| `author` | Your username |
| `category_id` | Must match the key used in `category_info.yaml` and `genre_info.yaml` |
| `variants` | Array of variant IDs matching the keys under your category in `genre_info.yaml` |

---

### Step 4: Define Category Settings (`category_info.yaml`)

This file controls how your category appears in Koi Box and which engines it uses:

```yaml
mypack:
  display_name: My Pack
  genre_selector_color: "8B2232"
  oled_color: "FF2244"
  album_cover_directory: mypack
  generator: profile
  generator_profile: mypack-gen
  lyrics_engine: llm          # "llm" for dynamic lyrics, omit for static/instrumental
  lyrics_config:               # Only needed if lyrics_engine: llm
    system_prompt_file: mypack_lyrics.txt
    language: any
    max_chars: 30
    placeholder_lyrics:
      - Placeholder line one
      - Placeholder line two
  bio_config:                  # Optional: LLM-generated artist bios
    system_prompt_file: mypack_bios.txt
    max_tokens: 200
```

| Field | Description |
|-------|-------------|
| `display_name` | Name shown in the Koi Box UI |
| `genre_selector_color` | Hex color for the genre selector (no `#` prefix) |
| `oled_color` | Brighter hex color used for OLED/dark mode |
| `album_cover_directory` | Subfolder name under `album_covers/` where images live |
| `generator` | Set to `profile` to use template-based name generation |
| `generator_profile` | Name of the YAML file in `generators/profiles/` (without `.yaml`) |
| `lyrics_engine` | Set to `llm` if you want the LLM to write lyrics dynamically. Omit for static lyrics or instrumentals. |
| `lyrics_config` | Configuration for LLM lyrics generation (see below) |
| `bio_config` | Configuration for LLM artist bio generation (optional) |

**lyrics_config fields:**
- `system_prompt_file` — Filename in `generators/prompts/` containing the LLM system prompt
- `language` — Language constraint for generated lyrics (e.g., `korean`, `english`, `any`)
- `max_chars` — Maximum characters per lyric line
- `placeholder_lyrics` — Fallback lyrics used if the LLM is unavailable

---

### Step 5: Define Your Variants (`genre_info.yaml`)

This is the heart of your pack. Each variant defines a complete set of instructions for ACE-Step to generate a specific style of music.

```yaml
mypack:
  variants:
    variant-one:
      description: >
        A human-readable description of what this variant sounds like.
        Not sent to ACE-Step — this is for documentation and UI tooltips.
      prefix: variantone
      caption: >-
        genre tags, instruments, vocal style, production qualities,
        mood descriptors — comma-separated tags that ACE-Step uses
        to shape the overall sound
      dynamic_lyrics: true
      lyrics_guidance: >-
        MOOD: Description of the emotional tone.
        THEMES: Core themes the lyrics should explore.
        IMAGERY: Specific images and metaphors to draw from.
      lyrics: |
        [Intro - description of intro sound]

        [Verse 1 - vocal style, energy level]
        {THEME_SEED}
        {LYRICS_VERSE_1_LINE_2}
        {LYRICS_VERSE_1_LINE_3}
        {LYRICS_VERSE_1_LINE_4}

        [Chorus - vocal style, energy level]
        {LYRICS_CHORUS_LINE_1}
        {LYRICS_CHORUS_LINE_2}
        {LYRICS_CHORUS_LINE_3}
        {LYRICS_CHORUS_LINE_4}

        [Outro - description of ending]
      bpm_min: 100
      bpm_max: 140
      key_scale: D Minor
      duration_min: 165
      duration_max: 240
      theme_seeds:
        - First theme seed line
        - Second theme seed line
        - ...
```

#### Key fields explained

**`caption`** — The most important field. This is the text prompt sent to ACE-Step that defines the overall sound. Write it as comma-separated tags covering:
- Genre/style (e.g., `alt rock, nu-metal, korean rock`)
- Instruments (e.g., `heavy distorted guitar riffs, drop-tuned guitar, punchy drums`)
- Vocal characteristics (e.g., `aggressive male vocals, emotional korean vocals`)
- Production qualities (e.g., `high energy, intense, raw, emotional`)
- Specific sonic details (e.g., `turntable scratches, gated reverb snare, synth stabs`)

See the [ACE-Step Tutorial's caption section](https://github.com/ace-step/ACE-Step-1.5/blob/main/docs/en/Tutorial.md) for detailed guidance on writing effective captions. Key principles:
- Be specific: `"sad piano ballad with female breathy vocal"` beats `"a sad song"`
- Combine dimensions: style + emotion + instruments + timbre anchors the sound
- Avoid conflicting terms (e.g., `"classical strings"` + `"hardcore metal"` in the same caption)
- Don't include BPM/key/tempo in the caption — those have dedicated fields

**`lyrics`** — The lyrics template sent to ACE-Step. This uses ACE-Step's structure tags (`[Verse]`, `[Chorus]`, `[Bridge]`, etc.) combined with modifier hints after a dash. Placeholders in `{CURLY_BRACES}` are filled in by the LLM or from `theme_seeds`.

Important ACE-Step lyrics principles:
- Structure tags tell the model what each section is and how to perform it
- Keep tag modifiers concise: `[Chorus - explosive, screamed]` not `[Chorus - anthemic - stacked harmonies - high energy - powerful - epic]`
- Keep lyrics and caption consistent — don't describe violin in the caption then use `[Guitar Solo]` in lyrics
- Aim for 6-10 syllables per line for natural rhythm
- Use UPPERCASE for high-intensity lines (e.g., screamed sections)
- Content in `(parentheses)` is treated as background vocals

**`prefix`** — A short string prepended to generated filenames for this variant. No spaces or special characters.

**`dynamic_lyrics`** — Set to `true` if you want the LLM to generate lyrics for the `{LYRICS_*}` placeholders. If `false` or omitted, the lyrics template is used as-is.

**`lyrics_guidance`** — Instructions sent to the LLM describing the mood, themes, and imagery for lyric generation. Only used when `dynamic_lyrics: true`.

**`bpm_min` / `bpm_max`** — BPM range. Koi Box picks a random value in this range for each song. Common ranges: slow (60-80), mid-tempo (90-120), fast (130-180). ACE-Step treats BPM as guidance, not an exact command.

**`key_scale`** — Musical key (e.g., `D Minor`, `C Major`, `Am`). Common keys (C, G, D, Am, Em) are most reliable in ACE-Step.

**`duration_min` / `duration_max`** — Song duration range in seconds. 150-240 seconds (2:30-4:00) is a typical range. Very long durations may cause repetition.

**`theme_seeds`** — A pool of opening lines or thematic seeds. One is randomly selected per song and placed at the `{THEME_SEED}` placeholder. These set the emotional direction for each track. Write 25-50+ seeds for good variety.

#### Lyrics placeholder naming

The placeholder names in your lyrics template must match what your LLM lyrics prompt expects to generate. Common patterns:

| Placeholder | Typical Use |
|-------------|-------------|
| `{THEME_SEED}` | Randomly selected from `theme_seeds` list |
| `{LYRICS_VERSE_1_LINE_2}` through `{LYRICS_VERSE_1_LINE_4}` | Verse lines |
| `{LYRICS_CHORUS_LINE_1}` through `{LYRICS_CHORUS_LINE_4}` | Chorus lines |
| `{LYRICS_RAP_LINE_1}` through `{LYRICS_RAP_LINE_4}` | Rap verse lines |
| `{LYRICS_BRIDGE_LINE_1}`, `{LYRICS_BRIDGE_LINE_2}` | Bridge lines |
| `{LYRICS_PRECHORUS_LINE_1}`, `{LYRICS_PRECHORUS_LINE_2}` | Pre-chorus lines |
| `{LYRICS_BREAKDOWN_LINE_1}`, `{LYRICS_BREAKDOWN_LINE_2}` | Breakdown lines |

You can define any placeholder names you want — just ensure your LLM prompt's output format matches.

---

### Step 6: Create the Generator Profile

The generator profile defines templates for randomly generating track names, artist names, and album names. Create a YAML file in `config/generators/profiles/`:

```yaml
# my-genre — generator profile for My Genre category.
# Pool file paths resolve relative to pools/{category}/ then pools/_shared/.

track_names:
  templates:
    - "{emotion}"
    - "{adj} {noun}"
    - "{verb} the {noun}"
    - "{noun} of {noun2}"
  pools:
    emotion: emotions.txt
    adj: adjectives.txt
    noun: nouns.txt
    noun2: nouns.txt
    verb: verbs.txt

artist_names:
  templates:
    - "{prefix}{suffix}"
    - "{word} {word2}"
    - "{solo}"
  feat_chance: 0.15
  feat_templates:
    - "{name} (feat. {feat})"
  pools:
    prefix: artist_prefixes.txt
    suffix: artist_suffixes.txt
    word: artist_words.txt
    word2: artist_words2.txt
    solo: artist_solo.txt
    feat: artist_feat.txt

album_names:
  templates:
    - "{adj} {noun}"
    - "{noun} {noun2}"
    - "{article} {adj} {noun}"
    - "{noun}"
  pools:
    adj: album_adjectives.txt
    noun: album_nouns.txt
    noun2: album_nouns.txt
    article: album_articles.txt
```

**How it works:**
- Each `templates` list contains format strings with `{placeholder}` variables
- A template is randomly selected, then each placeholder is filled by drawing a random line from the corresponding pool file
- `feat_chance` is the probability (0.0-1.0) that a generated artist name gets a "feat." collaboration appended
- Two different placeholders can reference the same pool file (e.g., `noun` and `noun2` both draw from `nouns.txt` but get independent random picks)

---

### Step 7: Create Word Pool Files

Create one text file per pool in `config/generators/pools/<category_id>/`. Each file is a simple list with one entry per line:

**Example: `adjectives.txt`**
```
Broken
Burning
Fallen
Hollow
Shattered
Frozen
Bleeding
```

**Example: `artist_prefixes.txt`**
```
Zero
Black
Dead
Red
Ash
Iron
```

Tips for good word pools:
- **Match your genre's aesthetic** — K-Alt uses dark, visceral words; a lo-fi pack might use cozy, nostalgic words
- **20+ entries per pool** provides good variety
- **One word or short phrase per line** — no punctuation needed
- **Consider multiple pool files** for the same role (e.g., `artist_words.txt` and `artist_words2.txt`) if you want compound names where both parts should feel different

---

### Step 8: Add Album Cover Images

Place your album cover images in `album_covers/<category_id>/`. Koi Box randomly assigns one to each generated track.

- **Format:** JPG recommended
- **Naming:** Use a consistent naming scheme (e.g., `mypack-0001.jpg`, `mypack-0002.jpg`)
- **Quantity:** 50-200 images provides good variety without repeats
- **Source:** [Unsplash](https://unsplash.com/) is a good source for freely-licensed images. Make sure you have the rights to distribute any images you include.

---

### Step 9: (Optional) Create LLM Prompts for Dynamic Lyrics

If you set `lyrics_engine: llm` in `category_info.yaml`, you need to create system prompt files that tell the LLM how to generate lyrics.

#### Lyrics prompt (`generators/prompts/<category>_lyrics.txt`)

This prompt instructs the LLM to generate lyric lines that fill the `{LYRICS_*}` placeholders in your lyrics template. Key things to include:

1. **Role and style context** — What kind of lyricist the LLM should be
2. **Language rules** — What language, register, and syllable constraints to follow
3. **Section-specific guidance** — How verse, chorus, rap, bridge lines should differ
4. **Concrete examples** — Show the exact `SLOT_NAME: text` format you expect
5. **Output format** — Be explicit that you want only `KEY: value` lines, no markdown or explanation

**K-Alt example excerpt:**
```
You are a Korean lyricist specializing in alt-rock and nu-metal songwriting.

RULES:
1. Write all lyrics in Korean (한국어).
2. Each line should be 3 to 10 Korean syllable blocks (자).
3. VERSE lines: emotional, restrained, building tension.
4. CHORUS lines: explosive, raw, singable — open vowels that can be screamed.
5. RAP lines: dense, rhythmic, rapid-fire.
...

OUTPUT FORMAT — return ONLY lines in this exact format:
LYRICS_SLOT_NAME: (korean text)
```

#### Bio prompt (`generators/prompts/<category>_bios.txt`)

Optional. Creates fictional artist biographies displayed in the UI:

```
You are writing short fictional bios for [genre] artists.

RULES:
1. Write 2-3 sentences maximum.
2. The artist is fictional — invent a believable origin.
3. Reference real locations naturally.
4. Ground the bio in the [genre] scene.
5. Do NOT use the artist's name in the bio text.
6. Write in English.
```

---

### Step 10: Test Your Pack

1. Copy your pack directory into Koi Box's expected pack location (or symlink it)
2. Start Koi Box with ACE-Step running
3. Your new category should appear in the genre selector
4. Test each variant:
   - Does the caption produce the right sonic character?
   - Are the lyrics coherent and fitting?
   - Do the generated names feel right for the genre?
   - Do the BPM/key/duration ranges feel natural?

**Iteration tips:**
- The **caption** has the biggest impact on sound. Adjust it first.
- If the AI vocal style is wrong, add more specific vocal descriptors to the caption (e.g., `breathy female vocal` vs just `female vocal`)
- If lyrics don't match the music's energy, adjust the structure tag modifiers (e.g., `[Chorus - whispered]` vs `[Chorus - explosive]`)
- If songs feel too same-y, widen your BPM range, add more theme seeds, and diversify your lyrics template structures across variants
- Test with ACE-Step directly (its Gradio UI) to iterate on captions faster before updating your pack

---

### Step 11: Register Your Pack

To publish your pack in this repository, add an entry to `index.json`:

```json
{
  "packs": [
    {
      "id": "mypack",
      "user": "your-username",
      "name": "My Pack",
      "description": "Short description of the pack.",
      "version": "1.0.0",
      "path": "packs/your-username/my-pack",
      "variants": ["variant-one", "variant-two", "variant-three"]
    }
  ]
}
```

Then submit a pull request to this repository.

---

## Quick Reference: File Cheat Sheet

| File | Purpose |
|------|---------|
| `pack.manifest` | Pack identity and metadata (JSON) |
| `config/category_info.yaml` | UI display settings, generator and lyrics engine config |
| `config/genre_info.yaml` | Variant definitions: captions, lyrics templates, BPM/key/duration, theme seeds |
| `config/generators/profiles/*.yaml` | Name generation templates linking to word pools |
| `config/generators/pools/<id>/*.txt` | Word lists for random name generation (one word per line) |
| `config/generators/prompts/*.txt` | LLM system prompts for dynamic lyrics and bios |
| `album_covers/<id>/*.jpg` | Album cover images randomly assigned to tracks |

---

## ACE-Step Prompting Reference

These tips are drawn from the [ACE-Step 1.5 Tutorial](https://github.com/ace-step/ACE-Step-1.5/blob/main/docs/en/Tutorial.md) and are most relevant when writing your `caption` and `lyrics` fields.

### Caption Tips
- **Be specific:** `"dreamy lo-fi hip hop, vinyl crackle, mellow female vocal, Rhodes piano, tape hiss"` > `"chill music"`
- **Combine dimensions:** Style + emotion + instruments + timbre + production
- **Avoid conflicts:** Don't combine contradictory styles (e.g., `"classical strings, hardcore metal"`)
- **Use texture words:** `warm`, `crisp`, `airy`, `punchy`, `raw`, `polished` influence mixing/timbre
- **Skip metadata in caption:** BPM, key, and duration have their own dedicated fields

### Lyrics Structure Tags
| Tag | Use |
|-----|-----|
| `[Intro]` | Opening section |
| `[Verse]` / `[Verse 1]` | Narrative verse |
| `[Pre-Chorus]` | Build energy before chorus |
| `[Chorus]` | Emotional peak |
| `[Bridge]` | Transition or contrast |
| `[Breakdown]` | Stripped-back intensity |
| `[Outro]` | Closing section |
| `[Instrumental]` | No vocals |
| `[Guitar Solo]`, `[Piano Interlude]` | Instrument features |

Add style hints with a dash: `[Chorus - explosive, screamed]`, `[Bridge - whispered, fragile]`

### Lyrics Writing Tips
- **6-10 syllables per line** for natural singability
- **UPPERCASE** for high-intensity/screamed delivery
- **(Parentheses)** for background vocals
- **Separate sections** with blank lines
- Keep one core emotion per song — don't scatter across unrelated themes

---

## License

[MIT](LICENSE)
