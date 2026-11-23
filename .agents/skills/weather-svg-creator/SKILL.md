---
name: weather-svg-creator
description: Creates an SVG weather card showing the current temperature and writes it to orchestration-workflow/weather.svg and orchestration-workflow/output.md
---

# Weather SVG Creator Skill

This skill creates an SVG weather card from provided temperature data.

## Task

Create an SVG weather card displaying the temperature for a given city and write it to the output files.

## Instructions

Use the temperature value, unit (Celsius or Fahrenheit), city name, and fetch time from the conversation. If the fetch failed or the temperature is missing, leave the existing outputs untouched.

### 1. Create SVG Weather Card

Generate a clean SVG weather card with the following structure:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 200" width="400" height="200">
  <rect width="400" height="200" rx="12" fill="#1a1a2e"/>
  <text x="200" y="50" text-anchor="middle" fill="#e0e0e0" font-family="system-ui" font-size="16">[City Name]</text>
  <text x="200" y="110" text-anchor="middle" fill="white" font-family="system-ui" font-size="48" font-weight="bold">[value]°[C/F]</text>
  <text x="200" y="150" text-anchor="middle" fill="#888" font-family="system-ui" font-size="12">[Fetch date and time]</text>
  <text x="200" y="185" text-anchor="middle" fill="#555" font-family="system-ui" font-size="10">Weather data from Open-Meteo API</text>
</svg>
```

Replace `[value]`, `[C/F]`, `[City Name]` with actual values from the input.

### 2. Write Output Files

Write the SVG content to `orchestration-workflow/weather.svg`.

Also write a markdown summary to `orchestration-workflow/output.md`:

```markdown
# Weather Report

![Weather Card](weather.svg)

**Location**: [City Name]
**Temperature**: [value]°[C/F]
**Fetched At**: [Fetch date and time in ISO 8601 format]

> Weather data provided by [Open-Meteo API](https://open-meteo.com/)
```

- Create the `orchestration-workflow/` directory if it does not exist
- Overwrite any existing content

## Expected Input

Temperature value, unit, and city from the calling agent:
```
Temperature: [X]°[C/F]
City: [City Name]
Unit: [Celsius/Fahrenheit]
Fetched At: [ISO 8601 timestamp]
```

## Expected Output

Two files written:
- `orchestration-workflow/weather.svg` — SVG weather card
- `orchestration-workflow/output.md` — Markdown report embedding the SVG

## Notes

- Use the exact temperature value and unit provided — do not re-fetch or modify the data
- Keep the SVG design minimal and clean
- Include a timestamp so the report is traceable
- Output files go in the `orchestration-workflow/` directory
