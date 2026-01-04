# FIFA World Cup Data Extraction Workflow

## Overview
This document describes the RPA workflow for extracting the first 10 rows of FIFA World Cup finals data from Wikipedia and appending it to a Google Sheet using the Google Sheets API.

**Source:** https://en.wikipedia.org/wiki/List_of_FIFA_World_Cup_finals

**Target:** Google Sheets (Spreadsheet ID: `1HsZjiP7f92hOt0v0FF7C_1umXoOBhLVcOjHjEZia5N8`)



## Workflow Steps

### Step 1: Open Page
**Step Name:** Open page  
**Input:** `https://en.wikipedia.org/wiki/List_of_FIFA_World_Cup_finals`  
**Output:** Wikipedia page loaded in browser  
**Notes:** This step begins an RPA browsing session



### Step 2: Parse Number
**Step Name:** Parse Number  
**Input:** String value: `"10"`  
**Output:** Integer value: `10`  
**Notes:** Converts the string "10" to integer 10 for use in the loop



### Step 3: Loop
**Step Name:** Loop  
**Input:** Number of iterations = 10 (parsed integer)  
**Output:** Index variable (values 1, 2, 3, ..., 10)  
**Notes:**
- Index variable starts from 1
- For 10 iterations, index will be 1 to 10
- If any error happens during the extraction steps inside the loop, the workflow skips to the next index value



### Step 4: Extract Year
**Step Name:** ExtractHTML  
**Input:**
```
//*[@id="mw-content-text"]/div[2]/table[4]/tbody/tr[{index}]/th/a
```
**Examples:**
- For index=1: `tr[1]` → extracts "1930"
- For index=2: `tr[2]` → extracts "1934"

**Output:** Year text (e.g., "1930", "1934", etc.)  
**Notes:**
- Captures text from Year column (in `<th>`)
- Stored in variable: `year_{index}`
- **On Error:** Skip to next iteration



### Step 5: Extract Winner
**Step Name:** ExtractHTML  
**Input:**
```
//*[@id="mw-content-text"]/div[2]/table[4]/tbody/tr[{index}]/td[1]/a
```
**Output:** Winner country name (e.g., "Uruguay", "Italy", etc.)  
**Notes:**
- Captures text from Winners column (second column, first `<td>`)
- Stored in variable: `winner_{index}`
- **On Error:** Skip to next iteration



### Step 6: Extract Score
**Step Name:** ExtractHTML  
**Input:**
```
//*[@id="mw-content-text"]/div[2]/table[4]/tbody/tr[{index}]/td[2]/a
```
**Output:** Score text (e.g., "4–2", "2–1", etc.)  
**Notes:**
- Captures text from Score column (third column, second `<td>`)
- The `<a>` tag contains the main score
- Stored in variable: `score_{index}`
- **On Error:** Skip to next iteration



### Step 7: Extract Runners-up
**Step Name:** ExtractHTML  
**Input:**
```
//*[@id="mw-content-text"]/div[2]/table[4]/tbody/tr[{index}]/td[3]/span/a
```
**Output:** Runners-up country name (e.g., "Argentina", "Czechoslovakia", etc.)  
**Notes:**
- Captures text from Runners-up column (fourth column, third `<td>`)
- Element nested inside `<span>` tag before the `<a>` tag
- Stored in variable: `runners_{index}`
- **On Error:** Skip to next iteration
- **On Success:** Continue to next iteration (if index < 10)



### Step 8: Call Google Sheets API
**Step Name:** Call API  
**Method:** POST  
**URL:**
```
https://sheets.googleapis.com/v4/spreadsheets/1HsZjiP7f92hOt0v0FF7C_1umXoOBhLVcOjHjEZia5N8:batchUpdate
```

**Headers:**
```
Content-Type: application/json
Authorization: Bearer {ACCESS_TOKEN}
```

**Request Body Structure:**
```json
{
  "requests": [
    {
      "appendCells": {
        "sheetId": 0,
        "rows": [
          {
            "values": [
              {"userEnteredValue": {"stringValue": "{year_1}"}},
              {"userEnteredValue": {"stringValue": "{winner_1}"}},
              {"userEnteredValue": {"stringValue": "{score_1}"}},
              {"userEnteredValue": {"stringValue": "{runners_1}"}}
            ]
          },
          {
            "values": [
              {"userEnteredValue": {"stringValue": "{year_2}"}},
              {"userEnteredValue": {"stringValue": "{winner_2}"}},
              {"userEnteredValue": {"stringValue": "{score_2}"}},
              {"userEnteredValue": {"stringValue": "{runners_2}"}}
            ]
          }
          // ... (repeat for all 10 rows)
        ],
        "fields": "userEnteredValue"
      }
    }
  ]
}
```

**Output:** API response confirming data appended to Google Sheet  
**Notes:**
- All 10 rows collected during loop
- Sent in a single batch API call after loop completes
- Uses `appendCells` request type within `batchUpdate` endpoint
- `sheetId: 0` refers to the first sheet (typically "Sheet1")
- `fields: "userEnteredValue"` specifies which cell properties to update



### Step 9: End
Workflow completes successfully.



## Complete API Call Example

### cURL Command:
```bash
curl --location 'https://sheets.googleapis.com/v4/spreadsheets/1HsZjiP7f92hOt0v0FF7C_1umXoOBhLVcOjHjEZia5N8:batchUpdate' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer ya29.a0Aa7pCA8oQer_AXg8Uic25gK4na5piC_r5lKhehifpEgLkD3xm1s543Q-bcMEnIAc18GgVkxXquvD6eA7XpvjoHi3i9JOpRyHrg2HTQnkeP6VWzpEDaVcH4kMzTdgec-73iduSYi9D3WL3f0mFJwUwmRriMYVia5zXE5gm0o81WOsMoXuAu9TkygrhrO_ECC3pIlb1_4aCgYKAXASARUSFQHGX2Mi__-s9pOu-Z_340ZpsbbiQw0206' \
--data '{
    "requests": [
        {
            "appendCells": {
                "sheetId": 0,
                "rows": [
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1930"}},
                            {"userEnteredValue": {"stringValue": "Uruguay"}},
                            {"userEnteredValue": {"stringValue": "4–2"}},
                            {"userEnteredValue": {"stringValue": "Argentina"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1934"}},
                            {"userEnteredValue": {"stringValue": "Italy"}},
                            {"userEnteredValue": {"stringValue": "2–1"}},
                            {"userEnteredValue": {"stringValue": "Czechoslovakia"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1938"}},
                            {"userEnteredValue": {"stringValue": "Italy"}},
                            {"userEnteredValue": {"stringValue": "4–2"}},
                            {"userEnteredValue": {"stringValue": "Hungary"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1950"}},
                            {"userEnteredValue": {"stringValue": "Uruguay"}},
                            {"userEnteredValue": {"stringValue": "2–1"}},
                            {"userEnteredValue": {"stringValue": "Brazil"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1954"}},
                            {"userEnteredValue": {"stringValue": "West Germany"}},
                            {"userEnteredValue": {"stringValue": "3–2"}},
                            {"userEnteredValue": {"stringValue": "Hungary"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1958"}},
                            {"userEnteredValue": {"stringValue": "Brazil"}},
                            {"userEnteredValue": {"stringValue": "5–2"}},
                            {"userEnteredValue": {"stringValue": "Sweden"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1962"}},
                            {"userEnteredValue": {"stringValue": "Brazil"}},
                            {"userEnteredValue": {"stringValue": "3–1"}},
                            {"userEnteredValue": {"stringValue": "Czechoslovakia"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1966"}},
                            {"userEnteredValue": {"stringValue": "England"}},
                            {"userEnteredValue": {"stringValue": "4–2"}},
                            {"userEnteredValue": {"stringValue": "West Germany"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1970"}},
                            {"userEnteredValue": {"stringValue": "Brazil"}},
                            {"userEnteredValue": {"stringValue": "4–1"}},
                            {"userEnteredValue": {"stringValue": "Italy"}}
                        ]
                    },
                    {
                        "values": [
                            {"userEnteredValue": {"stringValue": "1974"}},
                            {"userEnteredValue": {"stringValue": "West Germany"}},
                            {"userEnteredValue": {"stringValue": "2–1"}},
                            {"userEnteredValue": {"stringValue": "Netherlands"}}
                        ]
                    }
                ],
                "fields": "userEnteredValue"
            }
        }
    ]
}'
```

### Expected Response:
```json
{
    "spreadsheetId": "1HsZjiP7f92hOt0v0FF7C_1umXoOBhLVcOjHjEZia5N8",
    "replies": [
        {}
    ]
}
```



## Workflow Diagram Flow

```
Start
  ↓
[1] Open Page
  ↓
[2] Parse Number "10"
  ↓
[3] Loop (iterations: 1-10)
  ↓
  ├→ [4] Extract Year ──(error)──┐
  │    ↓ (success)               │
  ├→ [5] Extract Winner ─(error)─┤
  │    ↓ (success)               │
  ├→ [6] Extract Score ──(error)─┤
  │    ↓ (success)               │
  ├→ [7] Extract Runners-up ─────┘
  │    ↓ (success/error)
  │    └──→ Next iteration or back to [3]
  │
  ↓ (loop complete after 10 iterations)
[8] Call Google Sheets API (batchUpdate with appendCells)
  ↓
[9] End
```



## Key Features

### Error Handling
- Each extraction step (4-7) has error handling
- On error: Skip to next iteration without breaking the workflow
- Ensures workflow completes even if some rows fail

### Loop Logic
- Index starts at 1 and increments to 10
- XPath uses `tr[{index}]` to access correct table rows
- After iteration 10 completes (or on error), moves to API call step

### Data Collection
- All extracted data stored in variables during loop execution
- Variables follow pattern: `{field}_{index}` (e.g., `year_1`, `winner_1`)
- All 10 rows sent together in single API call after loop completes

### API Structure
- Uses `batchUpdate` endpoint with `appendCells` request
- Each row is a `RowData` object containing array of `CellData` objects
- Each cell uses `userEnteredValue` with `stringValue` property
- `sheetId: 0` targets the first sheet in the spreadsheet



## Data Extracted

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9c73be58-268e-47bf-ab2f-ddbc534d9f41" />

<img width="1894" height="964" alt="image" src="https://github.com/user-attachments/assets/a86402c0-7564-4072-9977-cca07c6b48c8" />




## Notes

- **XPath Strategy:** Uses table index 4 with hardcoded path for reliability
- **Row Indexing:** Loop index starts from 1, directly maps to table row index
- **API Authentication:** Requires valid OAuth 2.0 Bearer token in Authorization header
- **Sheet Target:** `sheetId: 0` appends data to the first sheet in the spreadsheet
- **Data Format:** All values sent as strings using `userEnteredValue.stringValue`
