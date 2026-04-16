# Power Automate Regular Expression Recipe Book
### The Complete Edition — Pattern Matching Without Regex

**Using:** String Functions · XPath & XML · Nested XPath · `join` & `concat` · `chunk` & `filter`

---

## Introduction

Power Automate and Azure Logic Apps do not natively support regular expressions. This recipe book provides battle-tested patterns to replicate the most common regex use cases using the expression functions available in the Workflow Definition Language.

Through a layered set of techniques — each building on the last — it is possible to cover approximately **70–75% of practical real-world regex use cases** without leaving the expression editor.

### The Five Technique Layers

| Layer | Functions | What It Unlocks |
|---|---|---|
| **1 — String Functions** | `startsWith`, `endsWith`, `contains`, `indexOf`, `substring`, `replace`, `split`, `trim`, `length`, `isInt`, `isFloat` | Anchors, presence, extraction, replacement |
| **2 — XPath + XML** | `xml()`, `xpath()`, `translate()` | Character classes, occurrence counting, whitespace detection, positional checks |
| **3 — Nested XPath** | Chained `xml()`/`xpath()` calls | Multi-segment validation, extract-then-validate, full format pipelines |
| **4 — join & concat** | `join()`, `concat()` | Set membership / alternation, data-driven whitelists, template matching |
| **5 — chunk & filter** | `chunk()`, `filter()` | Repeating sub-patterns, per-character scanning, uniform block validation |

### How to Read Each Recipe

Each recipe includes:
- **Regex Equivalent** — the regex pattern this replicates
- **Technique Layer** — which layer(s) are required
- **Description** — plain-English explanation
- **Scenario** — a concrete business use case
- **Expression** — the full Power Automate expression
- **Example** — input and expected output
- **Notes** — edge cases, limitations, or tips where relevant

---

> ⚠️ **Critical — Always Sanitise Input Before Wrapping in `xml()`**
>
> Any time you wrap a dynamic value in `xml(concat('<r>', value, '</r>'))`, first escape special XML characters. An unescaped `&`, `<`, or `>` in the input will cause the expression to fail at runtime.
>
> ```
> replace(replace(replace(trim(yourValue), '&', '&amp;'), '<', '&lt;'), '>', '&gt;')
> ```
>
> For postal codes, phone numbers, and other structured inputs this is unlikely to matter — but for any user-supplied free text it is essential.

---

> 💡 **XPath `substring()` is 1-indexed**
>
> Power Automate's native `substring(text, start, length)` is **0-indexed** (position 0 = first character).
> XPath's `substring(/r, position, length)` is **1-indexed** (position 1 = first character).
> Keep this in mind when mixing both in the same recipe.

---

## Contents

- [Beginner Recipes — String Functions](#beginner-recipes--string-functions) (Recipes 01–13)
- [Intermediate Recipes — XPath & XML](#intermediate-recipes--xpath--xml) (Recipes 14–23)
- [Advanced Recipes — Nested XPath](#advanced-recipes--nested-xpath) (Recipes 24–30)
- [Expert Recipes — join, concat, chunk & filter](#expert-recipes--join-concat-chunk--filter) (Recipes 31–40)
- [Real-World Recipes — Complete Worked Examples](#real-world-recipes--complete-worked-examples) (Recipes 41–44)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Capability Map — What Each Layer Unlocks](#capability-map--what-each-layer-unlocks)
- [What Remains Impossible](#what-remains-impossible)

---

# Beginner Recipes — String Functions

> These recipes use only Power Automate's built-in string functions. No `xml()` or `xpath()` required.

---

## Recipe 01 — Starts With Pattern
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^prefix` |

Tests whether a string begins with a specific fixed sequence of characters.

> 📋 **Scenario:** A support ticket system prefixes urgent tickets with `URG-`. Route any ticket whose ID starts with `URG-` to the high-priority queue.

```
startsWith(ticketId, 'URG-')
```

| Input | Returns |
|---|---|
| `ticketId = 'URG-00421'` | `true` |
| `ticketId = 'STD-00421'` | `false` |

---

## Recipe 02 — Ends With Pattern
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `suffix$` |

Tests whether a string ends with a specific fixed sequence. Useful for file extension checks and format suffixes.

> 📋 **Scenario:** A document management flow must only process PDF files. Check the filename ends with `.pdf` before passing it to the conversion action.

```
endsWith(toLower(fileName), '.pdf')
```

| Input | Returns |
|---|---|
| `fileName = 'Annual_Report_2024.PDF'` | `true` |
| `fileName = 'Budget.xlsx'` | `false` |

> 💡 Wrapping in `toLower()` catches `.PDF`, `.Pdf`, `.pdf` — effectively a case-insensitive suffix match.

---

## Recipe 03 — Contains Substring
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `.*keyword.*` |

Tests whether a string contains a specific substring anywhere within it.

> 📋 **Scenario:** A customer feedback flow should tag any message mentioning "refund" for escalation to the billing team.

```
contains(toLower(feedbackText), 'refund')
```

| Input | Returns |
|---|---|
| `feedbackText = 'I would like a Refund on my order'` | `true` |
| `feedbackText = 'Great service, thank you'` | `false` |

---

## Recipe 04 — Exact Full-String Match
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^exact_value$` |

Tests whether a string is exactly equal to a target value — nothing before or after.

> 📋 **Scenario:** A flow processes status codes from an API. Only proceed if the status is exactly `ACTIVE`.

```
equals(toUpper(trim(statusCode)), 'ACTIVE')
```

| Input | Returns |
|---|---|
| `statusCode = '  active  '` | `true` |
| `statusCode = 'INACTIVE'` | `false` |

> 💡 `trim()` removes accidental whitespace; `toUpper()` handles case variation.

---

## Recipe 05 — Empty String Check
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^$` |

Tests whether a string is empty or contains only whitespace.

> 📋 **Scenario:** A form submission flow must validate that the required "Company Name" field has not been left blank.

```
empty(trim(companyName))
```

| Input | Returns |
|---|---|
| `companyName = '   '` | `true` |
| `companyName = 'Contoso'` | `false` |

---

## Recipe 06 — Minimum / Maximum Length
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^.{min,max}$` |

Validates that a string's length falls within an acceptable range.

> 📋 **Scenario:** A user registration flow must ensure passwords are between 8 and 64 characters.

```
and(
  greaterOrEquals(length(password), 8),
  lessOrEquals(length(password), 64)
)
```

| Input | Returns |
|---|---|
| `password = 'Secur3P@ss'` | `true` |
| `password = 'abc'` | `false` |

---

## Recipe 07 — Anchored Both Ends
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^prefix.*suffix$` |

Validates that a string matches constraints at both ends simultaneously.

> 📋 **Scenario:** Internal invoice codes must start with `INV-` and end with the current year `-2024`.

```
and(
  startsWith(invoiceCode, 'INV-'),
  endsWith(invoiceCode, '-2024')
)
```

| Input | Returns |
|---|---|
| `invoiceCode = 'INV-ACME-0042-2024'` | `true` |
| `invoiceCode = 'PO-ACME-0042-2024'` | `false` |

---

## Recipe 08 — Extract After Delimiter
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `(?<=delimiter).*` |

Extracts everything after the first occurrence of a known delimiter.

> 📋 **Scenario:** Email addresses arrive as `user@domain.com`. Extract the domain portion for routing rules.

```
substring(
  emailAddress,
  add(indexOf(emailAddress, '@'), 1)
)
```

| Input | Returns |
|---|---|
| `emailAddress = 'jane.doe@contoso.com'` | `'contoso.com'` |

---

## Recipe 09 — Extract Before Delimiter
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `.*(?=delimiter)` |

Extracts everything before the first occurrence of a known delimiter.

> 📋 **Scenario:** Full names are stored as `FirstName LastName`. Extract just the first name for a personalised email greeting.

```
substring(
  fullName,
  0,
  indexOf(fullName, ' ')
)
```

| Input | Returns |
|---|---|
| `fullName = 'Jane Doe'` | `'Jane'` |

---

## Recipe 10 — Replace / Remove a Pattern
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `s/pattern/replacement/g` |

Replaces all occurrences of a fixed substring. Pass `''` as the replacement to delete it.

> 📋 **Scenario:** Phone numbers include hyphens and spaces. Strip all formatting to produce a clean dialable number.

```
replace(
  replace(trim(phoneNumber), '-', ''),
  ' ', ''
)
```

| Input | Returns |
|---|---|
| `phoneNumber = '0800-123 456'` | `'0800123456'` |

> 💡 Chain multiple `replace()` calls to remove more than one character type.

---

## Recipe 11 — Split on Delimiter
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `split(/delimiter/)` |

Splits a delimited string into an array of segments.

> 📋 **Scenario:** A product SKU follows the structure `CATEGORY-BRAND-MODEL`. Extract each component independently.

```
// Full array
split(sku, '-')

// First segment
first(split(sku, '-'))

// Second segment
split(sku, '-')[1]

// Last segment
last(split(sku, '-'))
```

| Input | Returns |
|---|---|
| `sku = 'ELEC-SONY-XM5'` | `['ELEC','SONY','XM5']` / `'ELEC'` / `'SONY'` / `'XM5'` |

---

## Recipe 12 — Integer-Only String
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `^\d+$` |

Tests whether an entire string represents a valid integer using the built-in `isInt()` function.

> 📋 **Scenario:** A data import flow receives customer IDs that must be positive integers. Reject any row where the field contains letters or symbols.

```
isInt(trim(customerId))
```

| Input | Returns |
|---|---|
| `customerId = '84729'` | `true` |
| `customerId = '847A9'` | `false` |

---

## Recipe 13 — Case-Insensitive Contains
`🟢 BEGINNER` · **Layer 1**

| | |
|---|---|
| **Regex Equivalent** | `/keyword/i` |

Performs a case-insensitive substring search by normalising both sides to the same case.

> 📋 **Scenario:** Detect the word "invoice" in email subjects regardless of capitalisation.

```
contains(toLower(emailSubject), 'invoice')
```

| Input | Returns |
|---|---|
| `emailSubject = 'RE: Your INVOICE #4421'` | `true` |
| `emailSubject = 'Meeting tomorrow'` | `false` |

---

# Intermediate Recipes — XPath & XML

> These recipes introduce `xml()` and `xpath()` to access XPath 1.0's `translate()` function — the key to character-class validation in Power Automate.

### The Core Technique

```
xpath(
  xml(concat('<r>', yourValue, '</r>')),
  'string-length(translate(/r, "characters_to_remove", "")) = 0'
)
```

`translate(string, remove, replace)` strips every character listed in the second argument. When the third argument is `""`, characters are deleted entirely. If nothing remains after stripping, the string contained only characters from the allowed set.

---

## Recipe 14 — All-Digits Character Class
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^[0-9]+$` |

Strips all digit characters using `translate()`. If nothing remains, the string was all digits.

> 📋 **Scenario:** A bank account number field must contain only digits before submitting to the payments API.

```
and(
  greater(length(trim(accountNumber)), 0),
  xpath(
    xml(concat('<r>', trim(accountNumber), '</r>')),
    'string-length(translate(/r, "0123456789", "")) = 0'
  )
)
```

| Input | Returns |
|---|---|
| `accountNumber = '12345678'` | `true` |
| `accountNumber = '1234X678'` | `false` |

---

## Recipe 15 — All-Letters Character Class
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^[a-zA-Z]+$` |

Strips all alphabetic characters. If nothing remains, the string contained only letters.

> 📋 **Scenario:** A "First Name" field should contain only letters — no digits, hyphens, or punctuation.

```
and(
  greater(length(trim(firstName)), 0),
  xpath(
    xml(concat('<r>', trim(firstName), '</r>')),
    'string-length(translate(/r,
      "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ",
      "")) = 0'
  )
)
```

| Input | Returns |
|---|---|
| `firstName = 'Katarina'` | `true` |
| `firstName = 'K4tarina'` | `false` |

---

## Recipe 16 — Alphanumeric-Only
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^[a-zA-Z0-9]+$` |

Combines letters and digits into the `translate()` removal set. Any remaining character reveals an invalid string.

> 📋 **Scenario:** Username fields must contain only letters and digits — no spaces, underscores, or special characters.

```
and(
  greater(length(trim(username)), 0),
  xpath(
    xml(concat('<r>', trim(username), '</r>')),
    'string-length(translate(/r,
      "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789",
      "")) = 0'
  )
)
```

| Input | Returns |
|---|---|
| `username = 'JaneDoe42'` | `true` |
| `username = 'Jane_Doe'` | `false` |

---

## Recipe 17 — Fixed Length + Digits
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^\d{5}$` |

Combines a `string-length` check with the all-digits test in a single XPath expression. The canonical approach for fixed-length numeric codes.

> 📋 **Scenario:** Validate a German postal code (Postleitzahl): exactly 5 digits, nothing else.

```
xpath(
  xml(concat('<r>', trim(postalCode), '</r>')),
  'string-length(/r) = 5 and
   string-length(translate(/r, "0123456789", "")) = 0'
)
```

| Input | Returns |
|---|---|
| `postalCode = '80331'` | `true` |
| `postalCode = '8033'` | `false` |
| `postalCode = '8033X'` | `false` |

---

## Recipe 18 — Count Character Occurrences
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `(char){n}` — occurrence count |

Counts how many times a specific character appears by comparing string length before and after removing it.

> 📋 **Scenario:** Validate that an email address contains exactly one `@` symbol.

```
equals(
  xpath(
    xml(concat('<r>', trim(emailAddress), '</r>')),
    'string-length(/r) - string-length(translate(/r, "@", ""))'
  ),
  1
)
```

| Input | Returns |
|---|---|
| `emailAddress = 'jane@contoso.com'` | `true` |
| `emailAddress = 'jane@@contoso.com'` | `false` |

---

## Recipe 19 — No Whitespace
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^\S+$` |

Checks that a string contains no whitespace at all — spaces, tabs, newlines, or carriage returns.

> 📋 **Scenario:** API keys must not contain any embedded spaces.

```
xpath(
  xml(concat('<r>', apiKey, '</r>')),
  'string-length(translate(/r, " &#9;&#10;&#13;", "")) = string-length(/r)'
)
```

| Input | Returns |
|---|---|
| `apiKey = 'abc123xyz'` | `true` |
| `apiKey = 'abc 123xyz'` | `false` |

> 💡 `&#9;` = tab · `&#10;` = newline · `&#13;` = carriage return

---

## Recipe 20 — Specific Character at Position
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^.{n-1}X` — character X at position n |

Validates the character at a specific positional index using XPath `substring()` (1-indexed).

> 📋 **Scenario:** A product code must have a hyphen as its 3rd character, e.g. `AB-12345`.

```
xpath(
  xml(concat('<r>', trim(productCode), '</r>')),
  'substring(/r, 3, 1) = "-"'
)
```

| Input | Returns |
|---|---|
| `productCode = 'AB-12345'` | `true` |
| `productCode = 'AB12345'` | `false` |

---

## Recipe 21 — Positional Character Class
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]{2}\d{4}$` |

Validates character class AND position by extracting slices with `substring()` then testing with `translate()`.

> 📋 **Scenario:** Country-year codes follow `DE2024` — exactly 2 uppercase letters then 4 digits.

```
and(
  equals(length(trim(countryYear)), 6),
  xpath(
    xml(concat('<r>', trim(countryYear), '</r>')),
    'string-length(translate(substring(/r, 1, 2),
      "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'
  ),
  xpath(
    xml(concat('<r>', trim(countryYear), '</r>')),
    'string-length(translate(substring(/r, 3, 4),
      "0123456789", "")) = 0'
  )
)
```

| Input | Returns |
|---|---|
| `countryYear = 'DE2024'` | `true` |
| `countryYear = '2024DE'` | `false` |

---

## Recipe 22 — Extract Middle Segment (Delimiter-Based Capture)
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `(?<=[^-]+-)[^-]+(?=-)` — second segment |

Uses `substring-after()` then `substring-before()` to extract content between the first and second delimiter occurrence.

> 📋 **Scenario:** Order references follow `REGION-ORDERID-YEAR`. Extract just the order ID.

```
xpath(
  xml(concat('<r>', orderRef, '</r>')),
  'substring-before(substring-after(/r, "-"), "-")'
)
```

| Input | Returns |
|---|---|
| `orderRef = 'EU-78432-2024'` | `'78432'` |

---

## Recipe 23 — Validate Segment Count
`🟡 INTERMEDIATE` · **Layer 2**

| | |
|---|---|
| **Regex Equivalent** | `^[^-]+-[^-]+-[^-]+$` — exactly 2 delimiters |

Counts delimiter occurrences to confirm the correct number of segments before attempting extraction.

> 📋 **Scenario:** Confirm an order reference has exactly 2 hyphens (3 segments) before extracting them.

```
equals(
  xpath(
    xml(concat('<r>', orderRef, '</r>')),
    'string-length(/r) - string-length(translate(/r, "-", ""))'
  ),
  2
)
```

| Input | Returns |
|---|---|
| `orderRef = 'EU-78432-2024'` | `true` |
| `orderRef = 'EU-78432'` | `false` |

---

# Advanced Recipes — Nested XPath

> These recipes nest one `xpath()` call inside another. The output of the inner expression becomes the input string to the outer `xml()`/`xpath()` pair — enabling multi-stage extract, transform, and validate pipelines with no intermediate variables.

### The Nesting Pattern

```
xpath(
  xml(concat('<r>',
    xpath(
      xml(concat('<r>', inputValue, '</r>')),
      'inner XPath expression'     ← extracts or transforms
    ),
  '</r>')),
  'outer XPath expression'         ← validates or extracts further
)
```

---

## Recipe 24 — Extract Then Validate
`🔴 ADVANCED` · **Layers 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | `^\w+-(\d{5})-\w+$` — validate a capture group |

Extracts a segment with an inner `xpath()` then immediately validates it with an outer `xpath()` — no variable needed.

> 📋 **Scenario:** Order references follow `REGION-POSTALCODE-SUFFIX`. Confirm the middle segment is a valid German postal code in one expression.

```
xpath(
  xml(concat('<r>',
    xpath(
      xml(concat('<r>', orderRef, '</r>')),
      'substring-before(substring-after(/r, "-"), "-")'
    ),
  '</r>')),
  'string-length(/r) = 5 and
   string-length(translate(/r, "0123456789", "")) = 0'
)
```

| Input | Returns |
|---|---|
| `orderRef = 'EU-80331-XYZ'` | `true` |
| `orderRef = 'EU-8033X-XYZ'` | `false` |

---

## Recipe 25 — Three-Segment Format Validation
`🔴 ADVANCED` · **Layers 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]{2}-\d{5}-[A-Z]{3}$` |

Validates all three segments of a structured code independently using nested XPath for each.

> 📋 **Scenario:** Validate a composite product code `DE-80331-ABC` — 2 uppercase letters, 5 digits, 3 uppercase letters.

```
and(
  // Guard: exactly 2 hyphens
  equals(
    xpath(xml(concat('<r>', code, '</r>')),
      'string-length(/r) - string-length(translate(/r, "-", ""))'),
    2
  ),

  // Segment 1: 2 uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', code, '</r>')),
        'substring-before(/r, "-")'),
    '</r>')),
    'string-length(/r) = 2 and
     string-length(translate(/r, "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'
  ),

  // Segment 2: 5 digits (extract via double nesting)
  xpath(
    xml(concat('<r>',
      xpath(
        xml(concat('<r>',
          xpath(xml(concat('<r>', code, '</r>')),
            'substring-after(/r, "-")'),
        '</r>')),
        'substring-before(/r, "-")'
      ),
    '</r>')),
    'string-length(/r) = 5 and
     string-length(translate(/r, "0123456789", "")) = 0'
  ),

  // Segment 3: 3 uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(
        xml(concat('<r>',
          xpath(xml(concat('<r>', code, '</r>')),
            'substring-after(/r, "-")'),
        '</r>')),
        'substring-after(/r, "-")'
      ),
    '</r>')),
    'string-length(/r) = 3 and
     string-length(translate(/r, "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'
  )
)
```

| Input | Returns |
|---|---|
| `code = 'DE-80331-ABC'` | `true` |
| `code = 'DE-80331-AB'` | `false` |
| `code = 'DE-8033X-ABC'` | `false` |

---

## Recipe 26 — Strip Prefix Then Validate Remainder
`🔴 ADVANCED` · **Layers 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | `^LITERAL([A-Z0-9]{6})$` |

Strips a known fixed prefix using `substring-after()`, then passes the remainder to a second validation layer.

> 📋 **Scenario:** License keys follow `LIC-XXXXXX` where X is any uppercase letter or digit. Validate that after stripping `LIC-` exactly 6 alphanumeric characters remain.

```
xpath(
  xml(concat('<r>',
    xpath(
      xml(concat('<r>', licenseKey, '</r>')),
      'substring-after(/r, "LIC-")'
    ),
  '</r>')),
  'string-length(/r) = 6 and
   string-length(translate(/r,
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789",
     "")) = 0'
)
```

| Input | Returns |
|---|---|
| `licenseKey = 'LIC-AB3X9Z'` | `true` |
| `licenseKey = 'LIC-AB3X9'` | `false` |
| `licenseKey = 'KEY-AB3X9Z'` | `false` |

---

## Recipe 27 — Count Occurrences Within Extracted Segment
`🔴 ADVANCED` · **Layers 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | Occurrence count within a capture group |

Extracts a segment with an inner `xpath()` then counts a specific character within it in the outer `xpath()`.

> 📋 **Scenario:** Version strings follow `PRODUCT-v1.2.3`. After extracting the version segment, confirm it contains exactly 2 dots (major.minor.patch format).

```
equals(
  xpath(
    xml(concat('<r>',
      xpath(
        xml(concat('<r>', versionString, '</r>')),
        'substring-after(/r, "-")'
      ),
    '</r>')),
    'string-length(/r) - string-length(translate(/r, ".", ""))'
  ),
  2
)
```

| Input | Returns |
|---|---|
| `versionString = 'MYAPP-v1.2.3'` | `true` |
| `versionString = 'MYAPP-v1.2'` | `false` |

---

## Recipe 28 — Extract Domain and Validate
`🔴 ADVANCED` · **Layers 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | `@([a-zA-Z0-9.]+)` |

Extracts the domain after `@` with an inner `xpath()`, then validates it contains only valid characters with the outer `xpath()`.

> 📋 **Scenario:** Before sending an automated email, confirm the domain part of the address contains only valid characters.

```
xpath(
  xml(concat('<r>',
    xpath(
      xml(concat('<r>', emailAddress, '</r>')),
      'substring-after(/r, "@")'
    ),
  '</r>')),
  'string-length(/r) > 0 and
   string-length(translate(/r,
     "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789.",
     "")) = 0'
)
```

| Input | Returns |
|---|---|
| `emailAddress = 'jane.doe@contoso.com'` | `true` |
| `emailAddress = 'jane.doe@contoso .com'` | `false` |

---

## Recipe 29 — Canadian Postal Code Validation
`🔴 ADVANCED` · **Layers 1 + 2**

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]\d[A-Z]\d[A-Z]\d$` |

Validates the alternating letter-digit pattern of Canadian postal codes by testing each of the 6 positions individually.

> 📋 **Scenario:** Confirm Canadian postal codes match the `A1A1A1` pattern before submitting to a postal API.

```
// Pre-processing — store as variable 'cleanCode'
toUpper(replace(postalCode, ' ', ''))

// Validation (using cleanCode)
and(
  equals(length(cleanCode), 6),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(/r,
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,1,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,2,1), "0123456789", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,3,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,4,1), "0123456789", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,5,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,6,1), "0123456789", "")) = 0')
)
```

| Input | `cleanCode` | Returns |
|---|---|---|
| `postalCode = 'M5V 3L9'` | `'M5V3L9'` | `true` |
| `postalCode = 'M5V 3L'` | `'M5V3L'` | `false` |
| `postalCode = '15V 3L9'` | `'15V3L9'` | `false` |

---

## Recipe 30 — Multi-Stage Normalise, Extract, Validate
`🔴 ADVANCED` · **Layers 1 + 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | Full pipeline: trim → normalise → extract → validate |

Combines string functions for pre-processing with nested XPath for validation. The pattern for dirty real-world data.

> 📋 **Scenario:** Product barcodes arrive inconsistently formatted (`ab - 12345 - xyz`). Normalise then validate against the expected pattern `AA-DDDDD-LLL`.

```
// Step 1: Normalise — store as variable 'clean'
toUpper(replace(replace(trim(rawBarcode), ' ', ''), '-', ''))
// 'ab - 12345 - xyz'  →  'AB12345XYZ'

// Step 2: Validate (using clean)
and(
  equals(length(clean), 10),

  // First 2 chars: uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 1, 2)'),
    '</r>')),
    'string-length(translate(/r, "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'
  ),

  // Middle 5 chars: digits
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 3, 5)'),
    '</r>')),
    'string-length(translate(/r, "0123456789", "")) = 0'
  ),

  // Last 3 chars: uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 8, 3)'),
    '</r>')),
    'string-length(translate(/r, "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'
  )
)
```

| Input | `clean` | Returns |
|---|---|---|
| `rawBarcode = 'ab - 12345 - xyz'` | `'AB12345XYZ'` | `true` |
| `rawBarcode = 'AB-12345-XY'` | `'AB12345XY'` | `false` |

---

# Expert Recipes — join, concat, chunk & filter

> These recipes use `join()`, `concat()`, `chunk()`, and `filter()` to unlock capabilities that require either runtime-constructed expressions or true iteration over string content.

---

## Recipe 31 — Exact Set Membership / Alternation
`⚫ EXPERT` · **Layer 4**

| | |
|---|---|
| **Regex Equivalent** | `^(RED\|GREEN\|BLUE)$` |

Tests whether a value is an exact member of a known set. The `|`-wrapping sentinel pattern ensures whole-value matching — a partial value like `DARKRED` will not match `RED`.

> 📋 **Scenario:** A status field must contain exactly one of a fixed set of allowed values: `PENDING`, `ACTIVE`, `CLOSED`, `CANCELLED`.

```
contains(
  concat('|', join(createArray('PENDING','ACTIVE','CLOSED','CANCELLED'), '|'), '|'),
  concat('|', toUpper(trim(statusValue)), '|')
)
```

| Input | Returns |
|---|---|
| `statusValue = 'active'` | `true` |
| `statusValue = 'DELETED'` | `false` |
| `statusValue = 'PROACTIVE'` | `false` |

> 💡 The `|` wrapping is critical. Without it, `PROACTIVE` would match `ACTIVE` via a simple substring search.

---

## Recipe 32 — Multi-Term Substring Search
`⚫ EXPERT` · **Layer 4**

| | |
|---|---|
| **Regex Equivalent** | `(error\|fail\|exception\|timeout)` |

Tests whether a string contains any of several substrings. For small fixed sets, chained `or(contains(...))` is fine. For larger or data-driven sets, build an XML node-set and use XPath `boolean()`.

> 📋 **Scenario:** Flag a log line as an error if it contains any of a configurable list of error keywords.

```
// Small fixed set — simple approach
or(
  contains(toLower(logLine), 'error'),
  contains(toLower(logLine), 'fail'),
  contains(toLower(logLine), 'exception'),
  contains(toLower(logLine), 'timeout')
)

// Data-driven approach — errorKeywords is an array variable
// ['error', 'fail', 'exception', 'timeout']
greater(
  length(
    filter(
      errorKeywords,
      contains(toLower(logLine), item())
    )
  ),
  0
)
```

| Input | Returns |
|---|---|
| `logLine = 'Connection timeout after 30s'` | `true` |
| `logLine = 'Request completed successfully'` | `false` |

---

## Recipe 33 — Data-Driven Character Whitelist
`⚫ EXPERT` · **Layers 4 + 5**

| | |
|---|---|
| **Regex Equivalent** | `^[allowed_chars]+$` with runtime-defined allowed set |

Instead of hardcoding the character set in a `translate()` XPath expression, use `chunk(input, 1)` + `filter()` to test each character against an allowed set stored in a variable. Makes validation rules configurable without rebuilding expressions.

> 📋 **Scenario:** An allowed character set for product codes is stored in a SharePoint configuration list and may change over time. Validate inputs against the current set without modifying the flow.

```
// allowedChars = ['A','B','C','D','E','F','0','1','2','3','4','5','6','7','8','9']
// (loaded from config at runtime)

equals(
  length(
    filter(
      chunk(trim(inputValue), 1),
      not(contains(join(allowedChars, ''), item()))
    )
  ),
  0
)
```

| Input | `allowedChars` | Returns |
|---|---|---|
| `inputValue = 'ABC123'` | `['A'-'F','0'-'9']` | `true` |
| `inputValue = 'ABC123G'` | `['A'-'F','0'-'9']` | `false` |

---

## Recipe 34 — Segment-Level Alternation
`⚫ EXPERT` · **Layers 1 + 4**

| | |
|---|---|
| **Regex Equivalent** | `^(A\|B\|C)(-(?:A\|B\|C))*$` — every segment from allowed set |

Splits a delimited string, then validates that every segment belongs to an allowed set. If the count of valid segments equals the total segment count, all segments are valid.

> 📋 **Scenario:** A colour sequence string like `RED-BLUE-GREEN-RED` must contain only recognised colour names separated by hyphens.

```
equals(
  length(
    filter(
      split(colourSequence, '-'),
      not(
        contains(
          concat('|RED|GREEN|BLUE|YELLOW|ORANGE|PURPLE|'),
          concat('|', toUpper(trim(item())), '|')
        )
      )
    )
  ),
  0
)
```

| Input | Returns |
|---|---|
| `colourSequence = 'RED-BLUE-GREEN'` | `true` |
| `colourSequence = 'RED-BLUE-VIOLET'` | `false` |

---

## Recipe 35 — Fixed-Width Repeating Sub-Pattern
`⚫ EXPERT` · **Layers 2 + 5**

| | |
|---|---|
| **Regex Equivalent** | `^(\d{2}[A-Z]){n}$` — n repetitions of a fixed-width pattern |

`chunk(input, n)` splits the string into fixed-width slices. `filter()` then validates every slice against the sub-pattern. If no slices fail, the whole string is valid. Works for **any n** as long as the chunk width is fixed.

> 📋 **Scenario:** A batch reference code is structured as pairs of 2-digits + 1 uppercase letter repeated an arbitrary number of times, e.g. `12A34B56C`.

```
and(
  // Length must be divisible by 3
  equals(mod(length(trim(batchCode)), 3), 0),

  // Every 3-character chunk must be \d{2}[A-Z]
  equals(
    length(
      filter(
        chunk(trim(batchCode), 3),
        not(
          and(
            xpath(xml(concat('<r>', item(), '</r>')),
              'string-length(translate(substring(/r,1,2),
               "0123456789","")) = 0'),
            xpath(xml(concat('<r>', item(), '</r>')),
              'string-length(translate(substring(/r,3,1),
               "ABCDEFGHIJKLMNOPQRSTUVWXYZ","")) = 0')
          )
        )
      )
    ),
    0
  )
)
```

| Input | Returns |
|---|---|
| `batchCode = '12A34B56C'` | `true` |
| `batchCode = '12A34B56'` | `false` — not divisible by 3 |
| `batchCode = '12A34b56C'` | `false` — lowercase letter |

---

## Recipe 36 — Uniform Block Validation
`⚫ EXPERT` · **Layers 2 + 5**

| | |
|---|---|
| **Regex Equivalent** | `^(\d{4})+$` — any number of 4-digit blocks |

Validates that a string consists entirely of uniform fixed-width blocks that each match the same pattern. The block-level equivalent of `+` quantifier on a fixed-width group.

> 📋 **Scenario:** A card number string must consist entirely of 4-digit blocks (with formatting already stripped), e.g. `1234567890123456`.

```
and(
  equals(mod(length(trim(cardNumber)), 4), 0),
  equals(
    length(
      filter(
        chunk(trim(cardNumber), 4),
        not(
          xpath(
            xml(concat('<r>', item(), '</r>')),
            'string-length(/r) = 4 and
             string-length(translate(/r, "0123456789", "")) = 0'
          )
        )
      )
    ),
    0
  )
)
```

| Input | Returns |
|---|---|
| `cardNumber = '1234567890123456'` | `true` |
| `cardNumber = '123456789012345'` | `false` — 15 digits, not divisible by 4 |
| `cardNumber = '1234567890123X56'` | `false` — contains letter |

---

## Recipe 37 — Per-Character Scan and Count
`⚫ EXPERT` · **Layers 4 + 5**

| | |
|---|---|
| **Regex Equivalent** | Count occurrences of a character — alternative to XPath method |

`chunk(input, 1)` exposes every character as an array element. `filter()` selects matching characters. `length()` counts them. An alternative to the XPath `translate()` occurrence-counting technique that works without `xml()`/`xpath()`.

> 📋 **Scenario:** Count how many times a specific character appears in a string — useful when the character might cause issues inside an XPath literal.

```
length(
  filter(
    chunk(input, 1),
    equals(item(), targetChar)
  )
)
```

| Input | `targetChar` | Returns |
|---|---|---|
| `input = 'hello world'` | `'l'` | `3` |
| `input = 'a,b,c,d,e'` | `','` | `4` |

---

## Recipe 38 — All Characters From Allowed Set (chunk method)
`⚫ EXPERT` · **Layers 4 + 5**

| | |
|---|---|
| **Regex Equivalent** | `^[allowed]+$` |

An alternative to the XPath `translate()` method for character class validation. Uses `chunk(1)` + `filter()` to check every character against a sentinel string. Preferred when the allowed set is dynamic or when avoiding XPath is simpler.

> 📋 **Scenario:** Validate that a hex colour code (after stripping `#`) contains only valid hexadecimal characters.

```
and(
  equals(length(trim(hexCode)), 6),
  equals(
    length(
      filter(
        chunk(toUpper(trim(hexCode)), 1),
        not(contains('0123456789ABCDEF', item()))
      )
    ),
    0
  )
)
```

| Input | Returns |
|---|---|
| `hexCode = 'FF5733'` | `true` |
| `hexCode = 'FF573G'` | `false` |
| `hexCode = 'FF573'` | `false` |

---

## Recipe 39 — Template Matching with Dynamic Length Bounds
`⚫ EXPERT` · **Layers 1 + 4**

| | |
|---|---|
| **Regex Equivalent** | `^FIXED-.{n,m}-FIXED$` — fixed anchors, variable middle |

Validates a template where the outer parts are known literals and the middle is variable-length, with configurable minimum and maximum lengths for the middle segment.

> 📋 **Scenario:** Invoice codes follow `INV-{reference}-2024` where the reference part must be between 3 and 10 characters.

```
and(
  startsWith(invoiceCode, 'INV-'),
  endsWith(invoiceCode, '-2024'),

  // Middle segment length check
  // Middle = strip 'INV-' from start and '-2024' from end
  greaterOrEquals(
    length(invoiceCode),
    add(length('INV--2024'), 3)     // minimum 3 chars in middle
  ),
  lessOrEquals(
    length(invoiceCode),
    add(length('INV--2024'), 10)    // maximum 10 chars in middle
  )
)
```

| Input | Returns |
|---|---|
| `invoiceCode = 'INV-ACME42-2024'` | `true` |
| `invoiceCode = 'INV-AC-2024'` | `false` — middle too short |
| `invoiceCode = 'INV-ACMECORPORATION-2024'` | `false` — middle too long |

---

## Recipe 40 — Left-Pad to Fixed Width Then Validate
`⚫ EXPERT` · **Layers 1 + 4**

| | |
|---|---|
| **Regex Equivalent** | `^\d{1,8}$` normalised to `^\d{8}$` |

Pads a variable-length numeric string to a canonical fixed-width form with leading zeros, then validates the padded result. Combines normalisation and validation in one pipeline.

> 📋 **Scenario:** Customer IDs arrive with variable leading zeros stripped (e.g. `4521` instead of `00004521`). Normalise to 8 digits for downstream system compatibility.

```
// Step 1: Validate the raw input is 1–8 digits
and(
  isInt(trim(customerId)),
  greaterOrEquals(length(trim(customerId)), 1),
  lessOrEquals(length(trim(customerId)), 8)
)

// Step 2: Pad to 8 digits — store as 'paddedId'
concat(
  substring('00000000', 0, sub(8, length(trim(customerId)))),
  trim(customerId)
)

// Step 3: Validate padded form is exactly 8 digits
xpath(
  xml(concat('<r>', paddedId, '</r>')),
  'string-length(/r) = 8 and
   string-length(translate(/r, "0123456789", "")) = 0'
)
```

| Input | `paddedId` | Returns |
|---|---|---|
| `customerId = '4521'` | `'00004521'` | `true` |
| `customerId = '123456789'` | n/a | `false` — too long |
| `customerId = '452A'` | n/a | `false` — not integer |

---

# Real-World Recipes — Complete Worked Examples

> These recipes tackle complete real-world scenarios using multiple technique layers together. Each represents a full multi-step solution as it would be implemented in a Power Automate flow.

---

## Recipe 41 — Extract All Email Addresses from Prose Text
`⚫ EXPERT` · **Layers 1 + 4**

| | |
|---|---|
| **Regex Equivalent** | `[^\s@]+@[^\s@]+\.[^\s@]+` — global match |

Extracts all email addresses embedded in a free-text string. Uses `@` as a reliable anchor: split on `@` to get fragments, extract the local part from the end of each pre-`@` fragment and the domain from the start of each post-`@` fragment.

> 📋 **Scenario:** Extract both email addresses from: `"...you can always reach me at anders@andersjensen.org, but...regards, Anders Jensen dontuse@thisemail.com"`

**How it works:**
Splitting on `@` produces N+1 fragments for N email addresses. The local part is the last whitespace-delimited word of each fragment (except the last), and the domain is the first whitespace-delimited word of each fragment (except the first), with trailing punctuation stripped.

```
// Input stored as variable: input

// ── Email 1 ──────────────────────────────────────────────────────
concat(
  // Local part: last word before first '@'
  last(split(trim(first(split(input, '@'))), ' ')),
  '@',
  // Domain: first word after first '@', comma stripped
  replace(
    first(split(split(input, '@')[1], ' ')),
    ',', ''
  )
)
// → 'anders@andersjensen.org'

// ── Email 2 ──────────────────────────────────────────────────────
concat(
  // Local part: last word of middle fragment
  last(split(trim(split(input, '@')[1]), ' ')),
  '@',
  // Domain: entire last fragment if no space, else first word
  if(
    equals(indexOf(split(input, '@')[2], ' '), -1),
    split(input, '@')[2],
    replace(first(split(split(input, '@')[2], ' ')), ',', '')
  )
)
// → 'dontuse@thisemail.com'

// ── As an array ───────────────────────────────────────────────────
createArray(
  concat(
    last(split(trim(first(split(input, '@'))), ' ')),
    '@',
    replace(first(split(split(input, '@')[1], ' ')), ',', '')
  ),
  concat(
    last(split(trim(split(input, '@')[1]), ' ')),
    '@',
    if(
      equals(indexOf(split(input, '@')[2], ' '), -1),
      split(input, '@')[2],
      replace(first(split(split(input, '@')[2], ' ')), ',', '')
    )
  )
)
```

| Input | Returns |
|---|---|
| `'...me at anders@andersjensen.org, but...Jensen dontuse@thisemail.com'` | `['anders@andersjensen.org', 'dontuse@thisemail.com']` |

> 💡 **Robustness notes:**
> - Email followed by comma: ✅ handled via `replace(..., ',', '')`
> - Email at end of string: ✅ handled via `indexOf(...) = -1` guard
> - More than 2 emails: ⚠️ extend the pattern for each additional `@` segment
> - `@` in non-email context: ⚠️ assumes all `@` characters belong to email addresses
> - Email followed by period (end of sentence): ⚠️ period would be included in domain — add additional `replace(..., '.', '')` guard if needed

---

## Recipe 42 — US Phone Number Validation
`⚫ EXPERT` · **Layers 1 + 2**

| | |
|---|---|
| **Regex Equivalent** | `^(\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}$` |

Validates a US phone number across all common formatting variants. Two-stage approach: normalise all formatting away, then validate the structural rules on the raw digits.

> 📋 **Scenario:** Accept US phone numbers in any common format from a web form and validate they represent a real dialable number.

**Accepted formats:** `2125551234` · `212-555-1234` · `(212) 555-1234` · `212.555.1234` · `+1 212 555 1234` · `1-212-555-1234` · `212 555 1234`

```
// ── Compose 1: normalised ─────────────────────────────────────────
// Strip all formatting characters
replace(replace(replace(replace(replace(replace(
  trim(phoneInput),
  '+', ''), ' ', ''), '-', ''), '.', ''), '(', ''), ')', '')

// ── Compose 2: clean ─────────────────────────────────────────────
// Strip leading country code '1' if 11 digits
if(
  and(
    equals(length(outputs('normalised')), 11),
    startsWith(outputs('normalised'), '1')
  ),
  substring(outputs('normalised'), 1),
  outputs('normalised')
)

// ── Condition: isValid ────────────────────────────────────────────
and(

  // Must be exactly 10 digits
  xpath(
    xml(concat('<r>', outputs('clean'), '</r>')),
    'string-length(/r) = 10 and
     string-length(translate(/r, "0123456789", "")) = 0'
  ),

  // Area code (position 1) must not start with 0 or 1
  not(
    xpath(
      xml(concat('<r>', outputs('clean'), '</r>')),
      'substring(/r, 1, 1) = "0" or substring(/r, 1, 1) = "1"'
    )
  ),

  // Exchange code (position 4) must not start with 0 or 1
  not(
    xpath(
      xml(concat('<r>', outputs('clean'), '</r>')),
      'substring(/r, 4, 1) = "0" or substring(/r, 4, 1) = "1"'
    )
  )

)
```

| Input | `clean` | Valid? |
|---|---|---|
| `212-555-1234` | `2125551234` | ✅ |
| `(212) 555-1234` | `2125551234` | ✅ |
| `+1 212 555 1234` | `2125551234` | ✅ |
| `1-212-555-1234` | `2125551234` | ✅ |
| `212.555.1234` | `2125551234` | ✅ |
| `0125551234` | `0125551234` | ❌ Area code starts with 0 |
| `1125551234` | `1125551234` | ❌ Area code starts with 1 |
| `2120551234` | `2120551234` | ❌ Exchange starts with 0 |
| `21255512` | `21255512` | ❌ Only 8 digits |
| `+44 207 555 1234` | `442075551234` | ❌ 12 digits |

**Optional — Exclude fictitious 555-01xx numbers:**
```
not(
  and(
    xpath(xml(concat('<r>', outputs('clean'), '</r>')),
      'substring(/r, 4, 3) = "555"'),
    xpath(xml(concat('<r>', outputs('clean'), '</r>')),
      'substring(/r, 7, 2) = "01"')
  )
)
```

---

## Recipe 43 — UK Postcode Validation
`⚫ EXPERT` · **Layers 1 + 2**

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]{1,2}\d[A-Z\d]?\s?\d[A-Z]{2}$` |

Validates UK postcodes across all six format variants by checking the shared structural rules that apply to all valid UK postcodes.

> 📋 **Scenario:** Validate UK postcodes submitted through an address form before geocoding.

**Valid formats:** `AN NAA` · `ANN NAA` · `AAN NAA` · `AANN NAA` · `ANA NAA` · `AANA NAA`

**Shared rules for all variants:**
- Total length 5–7 characters (space excluded)
- First character always a letter
- Last three characters always: digit + letter + letter
- Only alphanumeric characters

```
// Pre-processing — store as variable 'cleanCode'
toUpper(replace(postalCode, ' ', ''))

// Validation
and(
  greaterOrEquals(length(cleanCode), 5),
  lessOrEquals(length(cleanCode), 7),

  // Only alphanumeric characters
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(/r,
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", "")) = 0'),

  // First character is a letter
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r, 1, 1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  // Antepenultimate character (3rd from end) is a digit
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(
      substring(/r, string-length(/r) - 2, 1),
      "0123456789", "")) = 0'),

  // Last 2 characters are letters
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(
      substring(/r, string-length(/r) - 1, 2),
      "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0')
)
```

| Input | `cleanCode` | Returns |
|---|---|---|
| `SW1A 2AA` | `SW1A2AA` | ✅ |
| `M1 1AE` | `M11AE` | ✅ |
| `B1 1BB` | `B11BB` | ✅ |
| `EC1A 1BB` | `EC1A1BB` | ✅ |
| `12345` | `12345` | ❌ First char is digit |
| `SW1A 2A` | `SW1A2A` | ❌ Too short |

---

## Recipe 44 — Structured Reference Code: `AA-BBB-\d{1,6}`
`⚫ EXPERT` · **Layers 1 + 2 + 3**

| | |
|---|---|
| **Regex Equivalent** | `[A-Za-z]{2}-[A-Za-z]{3}-\d{1,6}` |

Validates a structured code where `AA` is any 2 characters, `BBB` is any 3 characters, and the final segment is 1–6 digits. Demonstrates the full validation pipeline for a pattern with variable-content segments.

> 📋 **Scenario:** Validate incoming reference codes from a partner system that follow a `XX-XXX-\d{1,6}` structure.

```
// Using split() to access each segment cleanly

and(
  // Exactly 2 hyphens
  equals(
    xpath(xml(concat('<r>', input, '</r>')),
      'string-length(/r) - string-length(translate(/r, "-", ""))'),
    2
  ),

  // Segment AA: exactly 2 characters
  equals(length(first(split(input, '-'))), 2),

  // Segment BBB: exactly 3 characters
  equals(length(split(input, '-')[1]), 3),

  // Segment digits: 1–6 digits
  and(
    greaterOrEquals(length(last(split(input, '-'))), 1),
    lessOrEquals(length(last(split(input, '-'))), 6),
    xpath(
      xml(concat('<r>', last(split(input, '-')), '</r>')),
      'string-length(translate(/r, "0123456789", "")) = 0'
    )
  )
)
```

| Input | Returns | Reason |
|---|---|---|
| `EU-ABC-1234` | ✅ | All segments valid |
| `EU-ABC-123456` | ✅ | 6-digit maximum |
| `EU-ABC-1` | ✅ | 1-digit minimum |
| `EU-AB-1234` | ❌ | BBB segment only 2 chars |
| `EU-ABC-1234567` | ❌ | 7 digits exceeds maximum |
| `EU-ABC-123X` | ❌ | Non-digit in final segment |
| `EUU-ABC-1234` | ❌ | AA segment is 3 chars |

> 💡 **Extraction variant:** To extract the numeric segment from a larger string where the code is embedded at a known position (e.g. always after `'Code: '`):
> ```
> // Extract candidate then validate
> last(split(
>   substring(input, add(indexOf(input, 'Code: '), 6)),
>   ' '
> )[0])
> ```
> For truly arbitrary embedded extraction (unknown position, no surrounding context), an Azure Function with a proper regex engine is the right tool.

---

# Quick Reference Cheat Sheet

| Regex Pattern | Power Automate Equivalent | Layer | Key Functions |
|---|---|---|---|
| `^prefix` | `startsWith(s, 'prefix')` | 1 | `startsWith` |
| `suffix$` | `endsWith(s, 'suffix')` | 1 | `endsWith` |
| `.*keyword.*` | `contains(s, 'keyword')` | 1 | `contains` |
| `^exact$` | `equals(trim(s), 'exact')` | 1 | `equals`, `trim` |
| `^$` | `empty(trim(s))` | 1 | `empty`, `trim` |
| `^.{n,m}$` | `and(greaterOrEquals(length(s),n), lessOrEquals(length(s),m))` | 1 | `length` |
| `^\d+$` | `isInt(s)` | 1 | `isInt` |
| `/keyword/i` | `contains(toLower(s), toLower(k))` | 1 | `toLower` |
| `s/foo/bar/g` | `replace(s, 'foo', 'bar')` | 1 | `replace` |
| `split(/d/)` | `split(s, 'd')` | 1 | `split` |
| `^[0-9]+$` | `xpath: translate digits = ''` | 2 | `xpath`, `translate` |
| `^[a-zA-Z]+$` | `xpath: translate letters = ''` | 2 | `xpath`, `translate` |
| `^[a-zA-Z0-9]+$` | `xpath: translate alphanum = ''` | 2 | `xpath`, `translate` |
| `^\d{n}$` | `xpath: length=n and translate digits = ''` | 2 | `xpath`, `translate` |
| `^\S+$` | `xpath: translate whitespace = length` | 2 | `xpath`, `translate` |
| `(char){n}` count | `length(s) - length(translate(s, char, ''))` | 2 | `xpath`, `translate` |
| `^.{n}X` | `xpath: substring(/r, n, 1) = 'X'` | 2 | `xpath`, `substring` |
| `^[A-Z]{2}\d{4}$` | `xpath: substring slice + translate per segment` | 2 | `xpath`, `translate`, `substring` |
| Extract after delimiter | `xpath: substring-after(/r, 'delim')` | 2 | `xpath`, `substring-after` |
| Extract before delimiter | `xpath: substring-before(/r, 'delim')` | 2 | `xpath`, `substring-before` |
| Extract 2nd segment | `xpath: substring-after then substring-before` | 2 | `xpath`, chained |
| `^\w+-(\d{5})-\w+$` validate | inner `xpath` extract + outer `xpath` validate | 3 | nested `xpath` |
| `^[A-Z]{2}-\d{5}-[A-Z]{3}$` | three nested `xpath` calls | 3 | nested `xpath` |
| `^PREFIX([A-Z0-9]{6})$` | `substring-after` then validate | 3 | nested `xpath` |
| `^(A\|B\|C)$` | `contains(concat('\|A\|B\|C\|'), concat('\|',v,'\|'))` | 4 | `concat`, `join` |
| `(err\|fail\|timeout)` | `filter(keywords, contains(s, item()))` | 4 | `filter`, `contains` |
| `^[dynamic_set]+$` | `filter(chunk(s,1), not(contains(join(set,''),item())))` | 4+5 | `chunk`, `filter`, `join` |
| `(\d{2}[A-Z]){n}` | `chunk(s,3)` + `filter` per-chunk validation | 5 | `chunk`, `filter` |
| `^(\d{4})+$` | `chunk(s,4)` + `filter` all-digit check | 5 | `chunk`, `filter` |

---

# Capability Map — What Each Layer Unlocks

| Regex Concept | Layer 1 | Layer 2 | Layer 3 | Layer 4 | Layer 5 |
|---|---|---|---|---|---|
| `^prefix` / `suffix$` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `.*keyword.*` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `^exact$` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `^.{n,m}$` length bounds | ✅ | ✅ | ✅ | ✅ | ✅ |
| `^\d+$` numeric string | ✅ | ✅ | ✅ | ✅ | ✅ |
| `/i` case-insensitive | ✅ | ✅ | ✅ | ✅ | ✅ |
| `[0-9]`, `[a-zA-Z]` char classes | ❌ | ✅ | ✅ | ✅ | ✅ |
| `^\d{n}$` fixed length + class | ❌ | ✅ | ✅ | ✅ | ✅ |
| `\s` / `\S` whitespace | ❌ | ✅ | ✅ | ✅ | ✅ |
| `(char){n}` occurrence count | ❌ | ✅ | ✅ | ✅ | ✅ |
| `^.{n}[A-Z]` positional class | ❌ | ✅ | ✅ | ✅ | ✅ |
| Delimiter-anchored capture | ❌ | ✅ | ✅ | ✅ | ✅ |
| Multi-segment format validation | ❌ | ⚠️ verbose | ✅ | ✅ | ✅ |
| Extract + validate in one pass | ❌ | ❌ | ✅ | ✅ | ✅ |
| `^(A\|B\|C)$` set membership | ❌ | ❌ | ❌ | ✅ | ✅ |
| Segment-level alternation | ❌ | ❌ | ❌ | ✅ | ✅ |
| Data-driven char whitelists | ❌ | ❌ | ❌ | ✅ | ✅ |
| Multi-term substring match | ❌ | ❌ | ❌ | ✅ | ✅ |
| `(\d{2}[A-Z]){n}` repeating sub-pattern | ❌ | ❌ | ❌ | ❌ | ✅ |
| `^(\d{4})+$` uniform blocks | ❌ | ❌ | ❌ | ❌ | ✅ |
| Per-character scanning | ❌ | ❌ | ❌ | ❌ | ✅ |

**Overall coverage:** Approximately **70–75% of practical real-world regex use cases.**

---

# What Remains Impossible

Even with all five technique layers combined, the following features have no solution within Power Automate's expression language. An **Azure Function** or **custom connector** exposing a real regex engine is the right tool for these cases.

| Feature | Example | Root Cause |
|---|---|---|
| **Backreferences** `\1` | `<([a-z]+)>.*</\1>` — tag name must match | Requires match state — no concept exists in PA expressions |
| **Lookaheads** `(?=...)` | `\d+(?=px)` — digits before "px" | Requires zero-width positional assertion |
| **Lookbehinds** `(?<=...)` | `(?<=\$)\d+` — digits after "$" | Requires zero-width positional assertion |
| **Negative lookahead** `(?!...)` | `foo(?!bar)` — "foo" not followed by "bar" | Requires zero-width positional assertion |
| **Consecutive identical chars** `(.)\1{2,}` | Detect `aaa`, `111` runs | `filter()` provides no index — cannot compare adjacent elements |
| **Recursive / balanced patterns** | `\(([^()]*\|(\(([^()]*)\)))*\)` | Requires recursion — not possible in any PA expression |
| **Conditional patterns** `(?(cond)yes\|no)` | Branch on whether group 1 matched | Requires branching within a single pattern |
| **Unicode property classes** `\p{L}` | Match any Unicode letter | XPath 1.0 has no Unicode properties — exhaustive enumeration impractical |
| **Variable-length arbitrary scan** | Find pattern at unknown position in unstructured text | Requires trying the pattern at every index — needs a loop |
| **Possessive quantifiers / atomic groups** | `\d++`, `(?>pattern)` | Backtracking control — no regex engine means no backtracking |

> 💡 **The fundamental boundary:** Almost every impossible feature shares the same root cause — regex engines are finite state machines that **scan input character by character, maintaining state, trying alternatives, and backtracking**. Power Automate expressions are **pure functions with no scan loop, no state, no backtracking, and no position cursor**. The five technique layers in this book push the boundary as far as it can go within that constraint — but they cannot change what expressions fundamentally are.

---

*Power Automate · Azure Logic Apps · Workflow Definition Language*
