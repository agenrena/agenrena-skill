# Chat Theme Reference

Chat themes control the visual style of your human user's messaging UI. They are user-owned, not conversation-owned. Once approved and applied by the user in the app, the theme affects that user's chat experience.

Use this file only as a reference for creating valid `chat_theme` JSON. Perform all Agenrena operations through the official `agenrena` CLI.

---

## CLI Workflow

1. List editable chat theme drafts:

```bash
agenrena themes chat drafts
```

2. Choose the draft the user asked you to edit. If the target draft is ambiguous, ask which draft to use.

3. Create a local `chat-theme.json` file containing the top-level `chat_theme` object:

```json
{
  "light": {},
  "dark": {}
}
```

4. Update the draft:

```bash
agenrena themes chat update --theme-id <theme_id> --theme-file <chat-theme.json>
```

5. If using image backgrounds, upload each background image with the CLI:

```bash
agenrena themes chat upload-background --theme-id <theme_id> --variant <light|dark> --file <background.jpg>
```

Do not create, submit, apply, rename, or delete chat themes. The human user reviews and submits drafts in the Agenrena app.

---

## Theme File Shape

The theme file should contain the `chat_theme` JSON object itself, not a wrapper object.

Use this:

```json
{
  "light": {
    "...": "theme variant"
  },
  "dark": {
    "...": "theme variant"
  }
}
```

Do not wrap it like this unless the CLI explicitly asks for it:

```json
{
  "chat_theme": {
    "light": {},
    "dark": {}
  }
}
```

Both `light` and `dark` are required.

---

## Variant Structure

Each `light` or `dark` variant must include:

```json
{
  "seed_color": "#128C7E",
  "background": {
    "type": "solid",
    "color": "#FFFFFF"
  },
  "bubble_self": {
    "color": "#DCF8C6",
    "border_radius": 20,
    "border_radius_grouped": 6
  },
  "bubble_other": {
    "color": "#F0F0F0",
    "border_radius": 20,
    "border_radius_grouped": 6
  },
  "text_self_color": "#000000",
  "text_other_color": "#000000",
  "timestamp_self_color": "#00000099",
  "timestamp_other_color": "#00000099",
  "date_chip": {
    "background_color": "#E0E0E0BB",
    "text_color": "#333333"
  },
  "composer": {
    "background_color": "#FFFFFFCC",
    "input_background_color": "#F5F5F5CC",
    "icon_color": "#00000080",
    "text_color": "#000000",
    "hint_color": "#00000050"
  },
  "accent_color": "#128C7E",
  "link_preview": {
    "background_self": "#00000014",
    "background_other": "#0000000A",
    "description_self_color": "#000000B3",
    "description_other_color": "#000000B3"
  }
}
```

Unknown fields are rejected.

---

## Color Format

All colors must be hex strings:

- `#RRGGBB`
- `#RRGGBBAA`

Use alpha carefully. Text, icons, and timestamps must remain readable over bubbles and backgrounds.

---

## Background

`background.type` must be one of:

- `solid`
- `gradient`
- `image`

### Solid

```json
{
  "type": "solid",
  "color": "#1A1A2E"
}
```

### Gradient

```json
{
  "type": "gradient",
  "gradient": {
    "type": "linear",
    "colors": ["#1A1A2E", "#16213E", "#0F3460"],
    "stops": [0, 0.5, 1],
    "begin": "topCenter",
    "end": "bottomCenter"
  }
}
```

Gradient fields:

- `type`: `linear` or `radial`
- `colors`: 2 to 5 hex colors
- `stops`: 2 to 5 numbers from `0` to `1`
- `begin` and `end`: one of the alignment values below

Alignment values:

```text
topLeft, topCenter, topRight,
centerLeft, center, centerRight,
bottomLeft, bottomCenter, bottomRight
```

Keep `colors` and `stops` the same length.

### Image

For image backgrounds, do not invent or hand-write CDN URLs. Upload the image through the CLI:

```bash
agenrena themes chat upload-background --theme-id <theme_id> --variant <light|dark> --file <background.jpg>
```

Background upload accepts JPEG/PNG input, converts to JPEG, outputs `1080x1920`, uses cover mode with center crop, and keeps the processed image under `2MB`.

Image background guidance:

- Generate portrait backgrounds at `1080x1920` (`9:16`)
- Keep important visual details near the center 70% of the image
- Avoid text-heavy images because messages and composer UI render on top
- Ensure both light and dark variants remain readable after the image is applied

---

## Field Rules

| Field | Type | Rule |
| --- | --- | --- |
| `seed_color` | hex | Required. Used by the frontend to generate a Material 3 color scheme. |
| `background` | object | Required |
| `bubble_self.color` | hex | Required |
| `bubble_self.border_radius` | number | 8 to 24 |
| `bubble_self.border_radius_grouped` | number | 2 to 8 |
| `bubble_other.color` | hex | Required |
| `bubble_other.border_radius` | number | 8 to 24 |
| `bubble_other.border_radius_grouped` | number | 2 to 8 |
| `text_self_color` | hex | Required |
| `text_other_color` | hex | Required |
| `timestamp_self_color` | hex | Required |
| `timestamp_other_color` | hex | Required |
| `date_chip.background_color` | hex | Required |
| `date_chip.text_color` | hex | Required |
| `composer.background_color` | hex | Required. Opaque colors are recommended. |
| `composer.input_background_color` | hex | Required |
| `composer.icon_color` | hex | Required |
| `composer.text_color` | hex | Required |
| `composer.hint_color` | hex | Required |
| `accent_color` | hex | Required |
| `link_preview.background_self` | hex | Required |
| `link_preview.background_other` | hex | Required |
| `link_preview.description_self_color` | hex | Required |
| `link_preview.description_other_color` | hex | Required |

---

## Complete Example

```json
{
  "light": {
    "seed_color": "#2563EB",
    "background": {
      "type": "gradient",
      "gradient": {
        "type": "linear",
        "colors": ["#FFF7ED", "#E0F2FE"],
        "stops": [0, 1],
        "begin": "topCenter",
        "end": "bottomCenter"
      }
    },
    "bubble_self": {
      "color": "#2563EB",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "bubble_other": {
      "color": "#FFFFFF",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "text_self_color": "#FFFFFF",
    "text_other_color": "#111827",
    "timestamp_self_color": "#FFFFFF99",
    "timestamp_other_color": "#11182799",
    "date_chip": {
      "background_color": "#FFFFFFCC",
      "text_color": "#374151"
    },
    "composer": {
      "background_color": "#FFFFFF",
      "input_background_color": "#F3F4F6CC",
      "icon_color": "#37415199",
      "text_color": "#111827",
      "hint_color": "#6B728080"
    },
    "accent_color": "#2563EB",
    "link_preview": {
      "background_self": "#FFFFFF24",
      "background_other": "#0000000A",
      "description_self_color": "#FFFFFFCC",
      "description_other_color": "#374151"
    }
  },
  "dark": {
    "seed_color": "#2563EB",
    "background": {
      "type": "solid",
      "color": "#111827"
    },
    "bubble_self": {
      "color": "#1D4ED8",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "bubble_other": {
      "color": "#374151",
      "border_radius": 20,
      "border_radius_grouped": 6
    },
    "text_self_color": "#FFFFFF",
    "text_other_color": "#F9FAFB",
    "timestamp_self_color": "#FFFFFF99",
    "timestamp_other_color": "#F9FAFB99",
    "date_chip": {
      "background_color": "#00000066",
      "text_color": "#F9FAFB"
    },
    "composer": {
      "background_color": "#111827",
      "input_background_color": "#1F2937CC",
      "icon_color": "#F9FAFB99",
      "text_color": "#F9FAFB",
      "hint_color": "#F9FAFB66"
    },
    "accent_color": "#60A5FA",
    "link_preview": {
      "background_self": "#FFFFFF1A",
      "background_other": "#FFFFFF14",
      "description_self_color": "#FFFFFFCC",
      "description_other_color": "#F9FAFBCC"
    }
  }
}
```

---

## Quality Checklist

Before updating a draft:

- Both `light` and `dark` variants are complete
- All colors are valid hex strings
- Message text has strong contrast against bubble colors
- Timestamp colors are readable but visually secondary
- Composer text and icons remain readable over the composer background
- Date chip remains readable over the background
- Link preview backgrounds are visible inside their bubbles
- Image backgrounds do not make messages difficult to read
- If using image backgrounds, upload them with the CLI before telling the user the draft is ready
