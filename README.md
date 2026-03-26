# FHIR Questionnaire Renderer

A zero-dependency, single-file web app that renders FHIR R4/R5 `Questionnaire` resources into fillable forms and generates spec-compliant `QuestionnaireResponse` output on submission.

No build step. No server. Open the HTML file in a browser and it works.

---

## Demo

1. Open `index.html` in any modern browser
2. Click **Load sample** to see a pre-built patient intake form
3. Fill out the form and click **Generate QuestionnaireResponse**
4. Copy or download the JSON output

---

## Features

### Input
- **Paste JSON** — drop raw Questionnaire JSON into the text area
- **File upload** — drag-and-drop or browse for a `.json` file
- Auto-detects **FHIR R4 vs R5** from the `fhirVersion` field

### Form rendering

Supports all common FHIR Questionnaire item types:

| Type | Rendered as |
|---|---|
| `string` | Text input |
| `text` | Multi-line textarea |
| `integer`, `decimal`, `quantity` | Number input |
| `boolean` | Yes / No toggle |
| `choice` | Custom radio (single) or checkbox (when `repeats: true`) |
| `open-choice` | Choice list + free-text "Other" option |
| `date`, `dateTime`, `time` | Native date/time pickers |
| `url` | Text input |
| `attachment` | File picker (base64-encoded in response) |
| `group` | Grouped card with section heading |
| `display` | Read-only info block |

Additional behaviors:
- `required: true` enforced on submit with inline field-level errors
- Progress bar tracks completion of required fields
- `repeats: true` on `choice` renders checkboxes and allows multi-select
- Sidebar shows questionnaire metadata: item count, required count, status, version

### Output
- Generates a FHIR-compliant `QuestionnaireResponse` with correct answer value types (`valueCoding`, `valueBoolean`, `valueDate`, `valueString`, etc.)
- Syntax-highlighted JSON preview
- **Copy to clipboard** or **Download as `.json`**

---

## Usage

No installation required.

```bash
# Option 1: Just open it
open index.html

# Option 2: Serve it locally if you prefer
npx serve .
# or
python3 -m http.server 8080
```

### Loading a questionnaire programmatically

The app expects a valid FHIR `Questionnaire` resource. Minimum required structure:

```json
{
  "resourceType": "Questionnaire",
  "id": "my-questionnaire",
  "fhirVersion": "4.0.1",
  "status": "active",
  "title": "My Form",
  "item": [
    {
      "linkId": "1",
      "text": "What is your name?",
      "type": "string",
      "required": true
    }
  ]
}
```

### FHIR version detection

The app reads `fhirVersion` from the Questionnaire root:

- `4.0.x` → **R4**
- `4.2.x`, `4.6.x`, `5.x` → **R5**

The generated `QuestionnaireResponse` is structured to match the detected version.

---

## QuestionnaireResponse output

A completed form produces a response like:

```json
{
  "resourceType": "QuestionnaireResponse",
  "status": "completed",
  "questionnaire": "Questionnaire/my-questionnaire",
  "authored": "2025-03-25T14:32:00.000Z",
  "item": [
    {
      "linkId": "1",
      "text": "What is your name?",
      "answer": [
        { "valueString": "Arjun Sharma" }
      ]
    }
  ]
}
```

Answer value types map directly to FHIR spec:

| Item type | Answer key |
|---|---|
| `string`, `text`, `url`, `open-choice` (free text) | `valueString` |
| `integer` | `valueInteger` |
| `decimal`, `quantity` | `valueDecimal` / `valueQuantity` |
| `boolean` | `valueBoolean` |
| `date` | `valueDate` |
| `dateTime` | `valueDateTime` |
| `time` | `valueTime` |
| `choice` (coded) | `valueCoding` |
| `attachment` | `valueAttachment` (base64) |

---

## Known limitations

- **`answerValueSet`** — items referencing external terminology servers render an empty choice list. A FHIR terminology endpoint (e.g. HAPI FHIR) would need to be wired in to resolve these.
- **`enableWhen`** — conditional display logic is not yet implemented. All items render regardless of conditions.
- **`answerConstraint`** — not enforced beyond `required`.
- Designed for single-patient / single-session use. No state persistence between sessions.

---

## Browser support

Any modern browser (Chrome, Firefox, Safari, Edge). No polyfills needed.

---

## File structure

```
index.html   # The entire app — HTML, CSS, JS in one file
README.md
```

---

## Roadmap

- [ ] `enableWhen` conditional logic
- [ ] `answerValueSet` with configurable FHIR terminology server
- [ ] `initial` value pre-population
- [ ] Multi-page questionnaires (`page` extension)
- [ ] Accessibility audit (ARIA labels, keyboard nav)
- [ ] FHIR validation of generated response against profile

---

## License

MIT
