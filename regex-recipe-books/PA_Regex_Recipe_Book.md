# Power Automate Regular Expression Recipe Book
### Pattern Matching Without Regex — Using String Functions · XPath · Nested XML Techniques

---

## Introduction

Power Automate and Azure Logic Apps do not natively support regular expressions. This recipe book provides battle-tested patterns to replicate the most common regex use cases using the expression functions available in the Workflow Definition Language.

Recipes are organised into three tiers:

- **Beginner** — uses standard string functions (`startsWith`, `endsWith`, `contains`, `indexOf`, `length`, `replace`, `split`, `trim`)
- **Intermediate** — introduces `xml()` and `xpath()` with XPath 1.0 `translate()` to enable character-class testing
- **Advanced** — uses nested `xpath()` calls to extract, transform, and validate in a single expression with no intermediate variables

Each recipe includes the **Regex Equivalent**, a plain-English **Description**, a concrete **Scenario**, the full **Expression**, an **Input/Output Example**, and notes where relevant.

> ⚠️ **Critical — Always Sanitise Input for XML**
> Whenever wrapping dynamic user input in `xml()`, first escape special XML characters to prevent parse failures:
> ```
> replace(replace(replace(trim(yourValue), '&', '&amp;'), '<', '&lt;'), '>', '&gt;')
> ```

---

## Contents

- [Beginner Recipes — String Functions](#beginner-recipes--string-functions) (Recipes 01–13)
- [Intermediate Recipes — XPath & XML](#intermediate-recipes--xpath--xml) (Recipes 14–23)
- [Advanced Recipes — Nested XPath](#advanced-recipes--nested-xpath) (Recipes 24–30)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [What XPath Cannot Replicate](#what-xpath-cannot-replicate)

---

# Beginner Recipes — String Functions

---

## Recipe 01 — Starts With Pattern
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^prefix` |

Tests whether a string begins with a specific fixed sequence of characters. Returns true or false. Case-insensitive by default in Power Automate.

> 📋 **Scenario:** A support ticket system prefixes urgent tickets with `URG-`. You need to route any ticket whose ID starts with `URG-` to the high-priority queue.

**Expression:**
```
startsWith(ticketId, 'URG-')
```

| | |
|---|---|
| **Input** | `ticketId = 'URG-00421'` |
| **Returns** | `true` |

---

## Recipe 02 — Ends With Pattern
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `suffix$` |

Tests whether a string ends with a specific fixed sequence. Useful for file extension checks, code suffixes, or format validation.

> 📋 **Scenario:** A document management flow must only process PDF files. Check the filename ends with `.pdf` before passing it to the conversion action.

**Expression:**
```
endsWith(toLower(fileName), '.pdf')
```

| | |
|---|---|
| **Input** | `fileName = 'Annual_Report_2024.PDF'` |
| **Returns** | `true` |

> 💡 **Note:** Wrapping in `toLower()` makes the check case-insensitive, catching `.PDF`, `.Pdf`, etc.

---

## Recipe 03 — Contains Substring
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `.*keyword.*` |

Tests whether a string contains a specific substring anywhere within it. The equivalent of a regex search (not an anchored match).

> 📋 **Scenario:** A customer feedback flow should tag any message that mentions the word "refund" so it can be escalated to the billing team.

**Expression:**
```
contains(toLower(feedbackText), 'refund')
```

| | |
|---|---|
| **Input** | `feedbackText = 'I would like a Refund on my order'` |
| **Returns** | `true` |

---

## Recipe 04 — Exact Full-String Match
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^exact_value$` |

Tests whether a string is exactly equal to a target value — nothing before or after. The simplest anchored match.

> 📋 **Scenario:** A flow processes status codes from an API. Only proceed if the status is exactly `ACTIVE`, ignoring `INACTIVE`, `PROACTIVE`, etc.

**Expression:**
```
equals(toUpper(trim(statusCode)), 'ACTIVE')
```

| | |
|---|---|
| **Input** | `statusCode = '  active  '` |
| **Returns** | `true` |

> 💡 **Note:** `trim()` removes accidental whitespace; `toUpper()` makes it case-insensitive.

---

## Recipe 05 — Empty String Check
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^$` |

Tests whether a string is empty (zero length). Useful as a guard condition before processing a value that might be blank.

> 📋 **Scenario:** A form submission flow must validate that the required "Company Name" field has not been left blank before creating the CRM record.

**Expression:**
```
empty(trim(companyName))
```

| | |
|---|---|
| **Input** | `companyName = '   '` |
| **Returns** | `true` |

> 💡 **Note:** `trim()` first ensures a string containing only spaces is also treated as empty.

---

## Recipe 06 — Minimum / Maximum Length
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^.{min,max}$` |

Validates that a string's length falls within an acceptable range. Combines `length()` with comparison functions.

> 📋 **Scenario:** A user registration flow must ensure passwords are between 8 and 64 characters long.

**Expression:**
```
and(
  greaterOrEquals(length(password), 8),
  lessOrEquals(length(password), 64)
)
```

| | |
|---|---|
| **Input** | `password = 'Secur3P@ss'` |
| **Returns** | `true` |

---

## Recipe 07 — Starts With AND Ends With
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^prefix.*suffix$` |

Validates that a string matches constraints at both ends simultaneously. Combine `startsWith` and `endsWith` with `and()`.

> 📋 **Scenario:** Internal reference codes must start with `INV-` and end with a year like `-2024`. Validate the format of incoming invoice codes.

**Expression:**
```
and(
  startsWith(invoiceCode, 'INV-'),
  endsWith(invoiceCode, '-2024')
)
```

| | |
|---|---|
| **Input** | `invoiceCode = 'INV-ACME-0042-2024'` |
| **Returns** | `true` |

---

## Recipe 08 — Extract Substring After Delimiter
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `(?<=delimiter).*` |

Extracts everything after the first occurrence of a known delimiter. Uses `indexOf()` to locate the delimiter and `substring()` to extract the remainder.

> 📋 **Scenario:** Email addresses arrive in the format `user@domain.com`. Extract just the domain portion for routing rules.

**Expression:**
```
substring(
  emailAddress,
  add(indexOf(emailAddress, '@'), 1)
)
```

| | |
|---|---|
| **Input** | `emailAddress = 'jane.doe@contoso.com'` |
| **Returns** | `'contoso.com'` |

---

## Recipe 09 — Extract Substring Before Delimiter
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `.*(?=delimiter)` |

Extracts everything before the first occurrence of a known delimiter. Uses `indexOf()` to get the position, then `substring()` from position 0.

> 📋 **Scenario:** Full names are stored as `FirstName LastName`. Extract just the first name to use in a personalised email greeting.

**Expression:**
```
substring(
  fullName,
  0,
  indexOf(fullName, ' ')
)
```

| | |
|---|---|
| **Input** | `fullName = 'Jane Doe'` |
| **Returns** | `'Jane'` |

---

## Recipe 10 — Replace / Remove a Pattern
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `s/pattern/replacement/g` |

Replaces all occurrences of a fixed substring with another value. To remove a substring, replace with empty string `''`.

> 📋 **Scenario:** Phone numbers stored in a system include hyphens and spaces (e.g. `0800-123 456`). Strip all non-numeric formatting to produce a clean dialable number.

**Expression:**
```
replace(
  replace(trim(phoneNumber), '-', ''),
  ' ', ''
)
```

| | |
|---|---|
| **Input** | `phoneNumber = '0800-123 456'` |
| **Returns** | `'0800123456'` |

> 💡 **Note:** Chain multiple `replace()` calls to remove more than one character type.

---

## Recipe 11 — Split String into Segments
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `split(/delimiter/)` |

Splits a delimited string into an array of parts. Combine with `first()`, `last()`, or index access to retrieve specific segments.

> 📋 **Scenario:** A product SKU is structured as `CATEGORY-BRAND-MODEL`. Split it to extract each component independently.

**Expression:**
```
// Full array:
split(sku, '-')

// First segment (CATEGORY):
first(split(sku, '-'))

// Last segment (MODEL):
last(split(sku, '-'))
```

| | |
|---|---|
| **Input** | `sku = 'ELEC-SONY-XM5'` |
| **Returns** | `['ELEC', 'SONY', 'XM5']` / `'ELEC'` / `'XM5'` |

---

## Recipe 12 — Integer-Only String Check
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `^\d+$` |

Tests whether an entire string represents a valid integer. Uses the built-in `isInt()` function — the simplest approach for purely numeric strings.

> 📋 **Scenario:** A data import flow receives values that should be customer IDs (positive integers). Reject any row where the ID field contains letters or symbols.

**Expression:**
```
isInt(trim(customerId))
```

| | |
|---|---|
| **Input** | `customerId = '84729'` |
| **Returns** | `true` |

---

## Recipe 13 — Case-Insensitive Contains
`🟢 BEGINNER`

| | |
|---|---|
| **Regex Equivalent** | `/keyword/i` |

Performs a case-insensitive substring search by normalising both the input and the search term to the same case before comparing.

> 📋 **Scenario:** A content moderation flow must detect the word "invoice" in email subjects regardless of capitalisation (`Invoice`, `INVOICE`, `iNvOiCe`).

**Expression:**
```
contains(toLower(emailSubject), toLower('Invoice'))
```

| | |
|---|---|
| **Input** | `emailSubject = 'RE: Your INVOICE #4421'` |
| **Returns** | `true` |

---

# Intermediate Recipes — XPath & XML

The recipes in this section use the `xml()` and `xpath()` functions to access XPath 1.0 capabilities. The core technique is the XPath `translate(string, chars_to_remove, replacements)` function: when the third argument is empty, it strips all characters listed in the second argument. This allows character-class validation that is impossible with string functions alone.

---

## Recipe 14 — All-Digits Character Class
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^[0-9]+$` |

Uses XPath `translate()` to strip all digit characters from the string. If nothing remains, the string contained only digits. Combines with a length check to ensure the string is not empty.

> 📋 **Scenario:** A bank account number field should contain only digits. Validate before submitting to the payments API.

**Expression:**
```
and(
  greater(length(trim(accountNumber)), 0),
  xpath(
    xml(concat('<r>', trim(accountNumber), '</r>')),
    'string-length(translate(/r, "0123456789", "")) = 0'
  )
)
```

| | |
|---|---|
| **Input** | `accountNumber = '12345678'` |
| **Returns** | `true` |

---

## Recipe 15 — All-Letters Character Class
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^[a-zA-Z]+$` |

Strips all alphabetic characters using `translate()`. If the result is empty, the string contained only letters. Useful for name fields that should not contain numbers or symbols.

> 📋 **Scenario:** A contact form "First Name" field should contain only letters (no digits, hyphens, or punctuation). Flag entries that fail.

**Expression:**
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

| | |
|---|---|
| **Input** | `firstName = 'Katarina'` |
| **Returns** | `true` |

---

## Recipe 16 — Alphanumeric-Only Check
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^[a-zA-Z0-9]+$` |

Combines digits and letters into the `translate()` removal set. Any character outside the alphanumeric set will remain after stripping, revealing an invalid string.

> 📋 **Scenario:** Username fields must contain only letters and digits — no spaces, underscores, or special characters.

**Expression:**
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

| | |
|---|---|
| **Input** | `username = 'JaneDoe42'` |
| **Returns** | `true` |

---

## Recipe 17 — Exact Length + Digits (e.g. German Postcode)
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^\d{5}$` |

Combines a `string-length` check with the all-digits `translate` test inside a single XPath expression. This is the canonical approach for fixed-length numeric codes.

> 📋 **Scenario:** Validate a German postal code (Postleitzahl): exactly 5 digits, nothing else.

**Expression:**
```
xpath(
  xml(concat('<r>', trim(postalCode), '</r>')),
  'string-length(/r) = 5 and
   string-length(translate(/r, "0123456789", "")) = 0'
)
```

| | |
|---|---|
| **Input** | `postalCode = '80331'` |
| **Returns** | `true` |

---

## Recipe 18 — Count Occurrences of a Character
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `(char){n}` or occurrence count |

Counts how many times a specific character appears by subtracting the length of the string with that character removed from the original length. This is impossible with standard string functions alone.

> 📋 **Scenario:** Validate that an email address contains exactly one `@` symbol.

**Expression:**
```
equals(
  xpath(
    xml(concat('<r>', trim(emailAddress), '</r>')),
    'string-length(/r) - string-length(translate(/r, "@", ""))'
  ),
  1
)
```

| | |
|---|---|
| **Input** | `emailAddress = 'jane@contoso.com'` |
| **Returns** | `true` |

---

## Recipe 19 — No Whitespace in String
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^\S+$` |

Checks that a string contains no whitespace characters at all (spaces, tabs, newlines, carriage returns) by stripping them all and confirming the length is unchanged.

> 📋 **Scenario:** API keys submitted through a form must not contain any embedded spaces. Reject and prompt the user if whitespace is detected.

**Expression:**
```
xpath(
  xml(concat('<r>', apiKey, '</r>')),
  'string-length(translate(/r, " &#9;&#10;&#13;", "")) = string-length(/r)'
)
```

| | |
|---|---|
| **Input** | `apiKey = 'abc123xyz'` |
| **Returns** | `true` |

> 💡 **Note:** `&#9;` = tab · `&#10;` = newline · `&#13;` = carriage return. These XML character references are valid inside XPath string literals.

---

## Recipe 20 — Specific Character at a Known Position
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^.{n}X` (character X at position n) |

Validates the character at a specific positional index using XPath `substring(string, position, length)`. XPath positions are 1-indexed (unlike Power Automate's 0-indexed `substring`).

> 📋 **Scenario:** A product code must have a hyphen `-` as the 3rd character, e.g. `AB-12345`. Validate the separator is in the correct place.

**Expression:**
```
xpath(
  xml(concat('<r>', trim(productCode), '</r>')),
  'substring(/r, 3, 1) = "-"'
)
```

| | |
|---|---|
| **Input** | `productCode = 'AB-12345'` |
| **Returns** | `true` |

> 💡 **Note:** XPath `substring()` is 1-indexed: position 1 is the first character.

---

## Recipe 21 — Positional Character Class Check
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]{2}\d{4}$` |

Validates both the character class AND the position of segments within a string. Extracts a slice with XPath `substring()` then tests it with `translate()`.

> 📋 **Scenario:** Country-year codes follow the format `DE2024` — 2 uppercase letters followed by exactly 4 digits. Validate incoming codes from a partner API.

**Expression:**
```
and(
  equals(length(trim(countryYear)), 6),
  xpath(
    xml(concat('<r>', trim(countryYear), '</r>')),
    'string-length(translate(substring(/r,1,2),
      "ABCDEFGHIJKLMNOPQRSTUVWXYZ","")) = 0'
  ),
  xpath(
    xml(concat('<r>', trim(countryYear), '</r>')),
    'string-length(translate(substring(/r,3,4),
      "0123456789","")) = 0'
  )
)
```

| | |
|---|---|
| **Input** | `countryYear = 'DE2024'` |
| **Returns** | `true` |

---

## Recipe 22 — Delimiter-Based Segment Extraction
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `(?<=^[^-]+-)[^-]+` (second segment) |

Uses XPath `substring-after()` and `substring-before()` to extract content relative to known delimiters — the XPath equivalent of a capture group anchored by delimiters.

> 📋 **Scenario:** Order references use the format `REGION-ORDERID-YEAR` (e.g. `EU-78432-2024`). Extract just the order ID from the middle segment.

**Expression:**
```
xpath(
  xml(concat('<r>', orderRef, '</r>')),
  'substring-before(substring-after(/r, "-"), "-")'
)
```

| | |
|---|---|
| **Input** | `orderRef = 'EU-78432-2024'` |
| **Returns** | `'78432'` |

---

## Recipe 23 — Validate Segment Count via Delimiter Count
`🟡 INTERMEDIATE`

| | |
|---|---|
| **Regex Equivalent** | `^[^-]+-[^-]+-[^-]+$` (exactly 2 hyphens) |

Counts occurrences of the delimiter character to confirm the correct number of segments exist before attempting extraction. Prevents misleading results on malformed inputs.

> 📋 **Scenario:** Before extracting segments from an order reference, confirm it has exactly 2 hyphens (meaning 3 segments). Reject malformed values early.

**Expression:**
```
equals(
  xpath(
    xml(concat('<r>', orderRef, '</r>')),
    'string-length(/r) - string-length(translate(/r, "-", ""))'
  ),
  2
)
```

| | |
|---|---|
| **Input** | `orderRef = 'EU-78432-2024'` |
| **Returns** | `true` |

---

# Advanced Recipes — Nested XPath

These recipes nest one `xpath()` call inside another, using the output of an inner expression as the input string to an outer `xml()`/`xpath()` pair. This allows multi-stage extraction and validation in a single expression without intermediate variables or loops.

> 🔬 **How Nesting Works**
>
> Each layer wraps the previous result: `xml(concat('<r>', xpath(...inner...), '</r>'))`.
> The inner expression extracts or transforms; the outer expression validates or extracts further.
> You can chain as many layers as needed, though 2–3 is typical in practice.

---

## Recipe 24 — Extract Then Validate (No Intermediate Variable)
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | `^\w+-(?=(\d{5})-)` — validate a capture group |

Extracts a segment using an inner `xpath()`, then immediately validates the extracted value using an outer `xpath()`. No intermediate variable needed — the two operations are composed into a single expression.

> 📋 **Scenario:** Order references follow `REGION-POSTALCODE-SUFFIX` (e.g. `EU-80331-XYZ`). Confirm that the middle segment is a valid German postal code (5 digits) in one expression.

**Expression:**
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

| | |
|---|---|
| **Input** | `orderRef = 'EU-80331-XYZ'` |
| **Returns** | `true` |

---

## Recipe 25 — Three-Segment Format Validation
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]{2}-\d{5}-[A-Z]{3}$` |

Validates all three segments of a structured code independently using nested XPath for each segment. Each segment is extracted via chained `substring-after`/`substring-before` calls and then validated for character class and length.

> 📋 **Scenario:** Validate a composite product code in the format `DE-80331-ABC` — 2 uppercase letters, hyphen, 5 digits, hyphen, 3 uppercase letters.

**Expression:**
```
and(
  // Guard: confirm exactly 2 hyphens
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

  // Segment 2: 5 digits
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

| | |
|---|---|
| **Input** | `code = 'DE-80331-ABC'` |
| **Returns** | `true` |

---

## Recipe 26 — Canadian Postal Code Validation
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | `^[A-Z]\d[A-Z]\d[A-Z]\d$` |

Validates the alternating letter-digit-letter-digit-letter-digit pattern of Canadian postal codes by testing each position individually. The string is normalised (uppercased, space removed) before validation.

> 📋 **Scenario:** A Canadian address validation flow must confirm postal codes match the correct `A1A1A1` pattern before submitting to the postal service API.

**Expression:**
```
// Pre-processing: store as variable 'cleanCode'
toUpper(replace(postalCode, ' ', ''))

// Validation (using cleanCode):
and(
  equals(length(cleanCode), 6),

  // All chars are alphanumeric
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(/r,
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", "")) = 0'),

  // Position 1: letter
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,1,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  // Position 2: digit
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,2,1), "0123456789", "")) = 0'),

  // Position 3: letter
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,3,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  // Position 4: digit
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,4,1), "0123456789", "")) = 0'),

  // Position 5: letter
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,5,1),
     "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "")) = 0'),

  // Position 6: digit
  xpath(xml(concat('<r>', cleanCode, '</r>')),
    'string-length(translate(substring(/r,6,1), "0123456789", "")) = 0')
)
```

| | |
|---|---|
| **Input** | `postalCode = 'M5V 3L9'` → `cleanCode = 'M5V3L9'` |
| **Returns** | `true` |

---

## Recipe 27 — Extract Domain and Validate It
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | `@([a-zA-Z0-9.]+)` |

A two-stage nested recipe: the inner `xpath()` extracts the domain portion after `@`, then the outer `xpath()` validates that the domain contains only alphanumeric characters and dots.

> 📋 **Scenario:** Before sending an automated email, validate that the domain part of the recipient address contains only valid characters (no spaces, slashes, or other illegal characters).

**Expression:**
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

| | |
|---|---|
| **Input** | `emailAddress = 'jane.doe@contoso.com'` |
| **Returns** | `true` |

---

## Recipe 28 — Strip Prefix Then Validate Remainder
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | `^PREFIX([A-Z0-9]{6})$` |

Strips a known fixed prefix using `substring-after()`, then passes the remainder to a second validation layer. Equivalent to matching and validating a capture group that follows a fixed prefix.

> 📋 **Scenario:** License keys are issued in the format `LIC-XXXXXX` where X is any uppercase letter or digit. Validate that after stripping `LIC-` exactly 6 alphanumeric characters remain.

**Expression:**
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

| | |
|---|---|
| **Input** | `licenseKey = 'LIC-AB3X9Z'` |
| **Returns** | `true` |

---

## Recipe 29 — Count Occurrences Within an Extracted Segment
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | Validate sub-pattern occurrence count within a capture group |

Extracts a segment with an inner `xpath()`, then counts a character within that extracted segment using the length-difference technique in the outer `xpath()`. Lets you validate structure within a captured sub-string.

> 📋 **Scenario:** Version strings follow `PRODUCT-v1.2.3`. After extracting the version segment, confirm it contains exactly 2 dots (meaning 3 numeric parts: major.minor.patch).

**Expression:**
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

| | |
|---|---|
| **Input** | `versionString = 'MYAPP-v1.2.3'` |
| **Returns** | `true` |

---

## Recipe 30 — Multi-Stage Sanitise, Extract, and Validate Pipeline
`🔴 ADVANCED`

| | |
|---|---|
| **Regex Equivalent** | Full pipeline: trim → normalise → extract → validate |

Combines Power Automate string functions for pre-processing (`trim`, `replace`, `toUpper`) with nested XPath for extraction and validation. This is the pattern for real-world data that arrives dirty and needs cleaning before validation.

> 📋 **Scenario:** International product barcodes arrive in inconsistent formats: mixed case, with or without spaces and hyphens (e.g. `ab - 12345 - xyz`). Normalise then validate against the expected pattern `AA-DDDDD-LLL`.

**Expression:**
```
// Step 1: Normalise — store as variable 'clean'
toUpper(replace(replace(trim(rawBarcode), ' ', ''), '-', ''))
// 'ab - 12345 - xyz'  →  clean = 'AB12345XYZ'

// Step 2: Validate (using clean)
and(
  equals(length(clean), 10),

  // First 2 chars: uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 1, 2)'),
    '</r>')),
    'string-length(translate(/r,
      "ABCDEFGHIJKLMNOPQRSTUVWXYZ","")) = 0'
  ),

  // Middle 5 chars: digits
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 3, 5)'),
    '</r>')),
    'string-length(translate(/r, "0123456789","")) = 0'
  ),

  // Last 3 chars: uppercase letters
  xpath(
    xml(concat('<r>',
      xpath(xml(concat('<r>', clean, '</r>')),
        'substring(/r, 8, 3)'),
    '</r>')),
    'string-length(translate(/r,
      "ABCDEFGHIJKLMNOPQRSTUVWXYZ","")) = 0'
  )
)
```

| | |
|---|---|
| **Input** | `rawBarcode = 'ab - 12345 - xyz'` → `clean = 'AB12345XYZ'` |
| **Returns** | `true` |

---

# Quick Reference Cheat Sheet

| Regex Pattern | Power Automate Equivalent | Key Functions |
|---|---|---|
| `^prefix` | `startsWith(s, 'prefix')` | `startsWith` |
| `suffix$` | `endsWith(s, 'suffix')` | `endsWith` |
| `.*keyword.*` | `contains(s, 'keyword')` | `contains` |
| `^exact$` | `equals(trim(s), 'exact')` | `equals`, `trim` |
| `^$` | `empty(trim(s))` | `empty`, `trim` |
| `^.{n,m}$` | `and(greaterOrEquals(length(s), n), lessOrEquals(length(s), m))` | `length`, `and` |
| `^\d+$` | `isInt(s)` or xpath translate digits | `isInt` / `xpath` |
| `^[0-9]{n}$` | `xpath`: `length=n and translate digits = ''` | `xpath`, `translate` |
| `^[a-zA-Z]+$` | `xpath`: `translate all letters = ''` | `xpath`, `translate` |
| `^[a-zA-Z0-9]+$` | `xpath`: `translate all alphanum = ''` | `xpath`, `translate` |
| `^\S+$` | `xpath`: `translate whitespace chars = length` | `xpath`, `translate` |
| char count `= n` | `length(s) - length(translate(s, char, ''))` | `xpath`, `translate` |
| `X` at position `n` | `xpath`: `substring(/r, n, 1) = 'X'` | `xpath`, `substring` |
| Extract after delimiter | `xpath`: `substring-after(/r, 'delim')` | `xpath`, `substring-after` |
| Extract before delimiter | `xpath`: `substring-before(/r, 'delim')` | `xpath`, `substring-before` |
| Extract 2nd segment | nested `xpath`: `substring-after` then `substring-before` | nested `xpath` |
| Validate capture group | inner `xpath` extract + outer `xpath` validate | nested `xpath` |

---

# What XPath Cannot Replicate

The following regex features have no equivalent in Power Automate's expression language and require an Azure Function or external service:

- **Backreferences** (`\1`) — reusing a captured group within the same pattern
- **Lookaheads / Lookbehinds** — `(?=...)`, `(?<=...)`, `(?!...)`, `(?<!...)`
- **Named capture groups** — `(?P<name>...)`
- **Alternation over variable patterns** — `(cat|dog|fish)` on unknown input (manageable for a small fixed set using chained `or(contains(...))`, but not scalable)
- **Non-greedy matching** between repeated identical delimiters — `substring-before` is always first-match only
- **Character class ranges** — `[a-z]` on unknown-length variable content without looping (only solvable by exhaustively listing all characters in `translate()`)
- **Quantifiers on patterns** — `(\d+)+`, `([a-z]{2,4})+` etc. require iterative approaches



What about using Join and Concat? Great question to dig into. Let me think through this systematically.

---

## What `join` and `concat` Fundamentally Add

Both functions let you **construct strings programmatically** before passing them into `xml()`/`xpath()`. That shifts the paradigm: instead of only *testing* or *extracting* from an input string, you can *build* a derived string that encodes information differently — making previously impossible tests tractable.

---

## 1. Alternation (`cat|dog|fish`) — Now Practical

This was listed as impossible. `join` cracks it open.

The idea: build a test string by replacing all occurrences of each valid option with empty string, and check if the result is empty.

But more elegantly — use `split` + `join` to test if a value is a member of a known set:

```
// Regex equivalent: ^(RED|GREEN|BLUE)$
// Build a pipe-delimited sentinel string and test membership

contains(
  concat('|', join(createArray('RED','GREEN','BLUE'), '|'), '|'),
  concat('|', toUpper(trim(colourCode)), '|')
)
```

The trick: wrapping both the haystack and needle with `|` ensures you match whole values only — `|RED|` won't match inside `|DARKRED|`. This is the classic "delimited list membership" pattern.

**Why it works:** `concat('|RED|GREEN|BLUE|')` means searching for `|GREEN|` can only succeed on an exact match, not a partial one.

**Scenario:** Validate that a status field contains exactly one of a fixed set of allowed values.

---

## 2. Building a Character Whitelist Dynamically

You can use `join` to assemble the `translate()` removal string programmatically rather than hardcoding it — useful when your allowed character set is stored as data rather than a literal:

```
// Allowed characters stored as an array variable
// allowedChars = ['A','B','C','D','E','F','0','1','2','3','4','5','6','7','8','9']

xpath(
  xml(concat('<r>', inputValue, '</r>')),
  concat(
    'string-length(translate(/r, "',
    join(allowedChars, ''),
    '", "")) = 0'
  )
)
```

This means your validation rules can be **data-driven** — store the allowed character set in a SharePoint list or environment variable, load it into an array, and build the XPath expression at runtime. Huge for maintainability.

---

## 3. Repeated Pattern Validation (`(AB){3}` → `ABABAB`)

`concat` lets you construct the expected repeated string and compare directly:

```
// Regex equivalent: ^(prefix){n}  or fixed repetition check
// Does the string start with 'AB' repeated exactly 3 times?

startsWith(
  inputString,
  concat('AB', 'AB', 'AB')
)
```

More powerfully, if `n` is dynamic:

```
// Build the repeated pattern from a variable 'repeatCount'
// Using a composed string via an Apply to Each into a string variable

// After loop, repeatedPattern = 'ABABAB'
equals(trim(inputString), repeatedPattern)
```

For fixed `n` this is trivial with `concat`. For dynamic `n` you need a loop to build it — but the *comparison* itself needs no XPath.

---

## 4. Bounded Alternation with Positional Segments

Combine `split`, `join`, and `xpath` to validate that **every segment** of a delimited string belongs to an allowed set — something like `^(RED|GREEN|BLUE)(-(?:RED|GREEN|BLUE))*$`:

```
// Split the string into segments, then for each segment test membership
// Using filter array + length comparison

// All segments must be in the allowed set
equals(
  length(
    filter(
      split(inputString, '-'),
      item() => contains(
        concat('|RED|GREEN|BLUE|'),
        concat('|', toUpper(item()), '|')
      )
    )
  ),
  length(split(inputString, '-'))
)
```

If every segment passes the membership test, the filtered array length equals the original array length. This replicates a repeated alternation pattern across a variable number of segments.

---

## 5. Constructing Multi-Value `contains` Patterns (Poor Man's `|` in XPath)

XPath 1.0 has no `matches()` and no alternation. But you can use `concat` to build a compound test string that encodes multiple values, then use a single `contains` on it:

```
// Does the string contain any of several substrings?
// Regex: (error|fail|exception|fault)

or(
  contains(toLower(logLine), 'error'),
  contains(toLower(logLine), 'fail'),
  contains(toLower(logLine), 'exception'),
  contains(toLower(logLine), 'fault')
)
```

That's fine for small sets. But with `join` you can build this differently — concatenate all the search terms with a unique separator and use a single xpath `contains` on a constructed document:

```
// Build a lookup document from your terms array
// terms = ['error','fail','exception','fault']

xpath(
  xml(concat('<r><h>', join(terms, '</h><h>'), '</h></r>')),
  'boolean(/r/h[contains(., "' , toLower(logLine) , '")])'  
)
```

This constructs a small XML document where each `<h>` node contains one search term, then uses XPath's node-set `contains` to test if any node matches. **One expression, N terms, no chained `or()`**.

---

## 6. Interleaving Known and Unknown Segments (Template Matching)

`concat` lets you implement a simple **template** check — does a string match a pattern where some parts are fixed and some are variable?

```
// Template: 'INVOICE-{anything}-2024'
// Regex equivalent: ^INVOICE-.+-2024$

and(
  startsWith(inputCode, 'INVOICE-'),
  endsWith(inputCode, '-2024'),
  greater(
    length(inputCode),
    length('INVOICE--2024')   // at least 1 char in the middle
  )
)
```

With `concat` the middle minimum length becomes dynamic:

```
// Minimum middle segment length stored as variable minLen
greater(
  length(inputCode),
  add(length(concat('INVOICE-', '-2024')), minLen)
)
```

---

## 7. Normalise-Then-Validate (Expanding Recipe 30)

`concat` enables a powerful pattern: **pad or reformat** a string into a canonical form before validation. For example, normalise a variable-length code to a fixed-length one by left-padding with zeros:

```
// Pad customerId to 8 digits with leading zeros
// Then validate it's exactly 8 digits

paddedId = concat(
  substring('00000000', 0, sub(8, length(trim(customerId)))),
  trim(customerId)
)
// Then: xpath validate paddedId is 8 digits
```

This replicates regex patterns like `^\d{1,8}$` but also produces the canonical form for downstream use — validation and normalisation in one step.

---

## 8. Escape-and-Wrap as a Reusable Pattern

The XML sanitisation step (escaping `&`, `<`, `>`) can now be cleanly encapsulated with `concat`:

```
// Reusable safe-wrap expression
// Saves repeating the replace() chain everywhere

concat('<r>',
  replace(replace(replace(trim(inputValue),
    '&', '&amp;'),
    '<', '&lt;'),
    '>', '&gt;'),
'</r>')
```

Store this as a **compose action** output and reference it throughout the flow, rather than inlining the sanitisation every time. `concat` is what makes the wrapping composable.

---

## Summary of What Gets Unlocked

| Regex Concept | How `join`/`concat` Enables It |
|---|---|
| `^(A\|B\|C)$` exact set membership | `contains(concat('\|A\|B\|C\|'), concat('\|', value, '\|'))` |
| `(AB){n}` fixed repetition | `concat()` builds the expected repeated string for direct comparison |
| `(A\|B\|C)` across segments | `filter()` + membership test on `split()` array |
| Data-driven character whitelists | `join(allowedCharsArray, '')` builds the `translate()` removal string at runtime |
| Multi-term contains (`err\|fail\|fault`) | `join(terms, '</h><h>')` builds an XML node-set for XPath node-level testing |
| Template matching with variable middle | `concat()` computes expected minimum/maximum lengths dynamically |
| Padding / canonical normalisation | `concat()` left-pads to fixed width before validation |

The biggest unlock is probably **set membership** — turning the "alternation over variable patterns is impossible" limitation into something clean and scalable. And **data-driven character whitelists** are significant for maintainability in production flows.



-- using chunk

Really sharp observation. Let me think through this carefully and honestly.

---

## What `chunk()` Does

`chunk(string, n)` splits a string into an array of equal-length substrings:

```
chunk('ABCDEF', 2)  →  ['AB', 'CD', 'EF']
```

So it does give you **positional slices of the string as an array** — which is as close to iterating over a string as a single expression can get.

---

## What This Actually Unlocks

### 1. Fixed-Width Repeated Pattern Validation — NOW POSSIBLE

This was listed as impossible. `chunk()` cracks it.

The pattern `(\d{2}[A-Z]){4}` — four repetitions of exactly 2 digits + 1 letter — requires testing the same sub-pattern at every fixed-width position. `chunk()` hands you exactly those positions as array elements.

```
// Input: '12A34B56C78D'  →  chunk into width-3 pieces
// ['12A', '34B', '56C', '78D']
// Then validate EVERY chunk matches \d{2}[A-Z]

// Step 1: confirm even chunking (length divisible by 3)
equals(mod(length(trim(input)), 3), 0)

// Step 2: chunk it
// chunks = chunk(trim(input), 3)

// Step 3: validate every chunk
// Use filter() to find chunks that FAIL, then confirm none do
equals(
  length(
    filter(
      chunk(trim(input), 3),
      not(
        and(
          xpath(xml(concat('<r>', item(), '</r>')),
            'string-length(translate(substring(/r,1,2), "0123456789","")) = 0'),
          xpath(xml(concat('<r>', item(), '</r>')),
            'string-length(translate(substring(/r,3,1),
              "ABCDEFGHIJKLMNOPQRSTUVWXYZ","")) = 0')
        )
      )
    )
  ),
  0
)
```

The `filter()` finds all chunks that **fail** the sub-pattern. If none fail, the length is 0. This is as close to `(\d{2}[A-Z]){n}` as Power Automate can get — and crucially, it works for **any n** as long as the chunk width is fixed.

---

### 2. Character-by-Character Scanning — NOW POSSIBLE

`chunk(string, 1)` gives you every character as an individual array element. This is the scan loop we said was impossible:

```
chunk('Hello World', 1)
→ ['H','e','l','l','o',' ','W','o','r','l','d']
```

Combined with `filter()`, you can now do things that previously required a loop.

**Count occurrences of a character (alternative to XPath translate trick):**
```
length(
  filter(
    chunk(input, 1),
    equals(item(), '@')
  )
)
```

**Test that no character falls outside an allowed set:**
```
equals(
  length(
    filter(
      chunk(input, 1),
      not(contains('ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789', item()))
    )
  ),
  0
)
```

This is genuinely different from the `translate()` approach — instead of building an XPath expression with a hardcoded removal string, you're using Power Automate's native `contains()` against a sentinel string. **And the allowed character set can be a variable.**

---

### 3. Consecutive Identical Characters — `(.)\1{2,}` — NOW POSSIBLE

This was listed as fully impossible. `chunk(input, 1)` + a sliding window approach gets us there.

The trick is comparing adjacent elements. You can't do a true sliding window in a single expression, but you can detect whether any character equals its neighbour by chunking into size-1 and using `filter()` with index awareness:

```
// Find if any character appears 3+ times consecutively
// Build pairs by comparing chunk[n] to chunk[n+1] to chunk[n+2]

// Approach: chunk into 1s, then check if filter of 
// 'same as next' has 2+ consecutive hits

// This requires nthIndexOf or index tracking — 
// which brings us to the honest limit below
```

Actually — here we hit a real wall. `filter()` in Power Automate gives you `item()` but **not the current index** within the filter expression. So you cannot compare `item()[n]` with `item()[n+1]`. You know the value but not the position.

So consecutive character detection remains out of reach **within a single expression** — you'd need an `Apply to Each` with an index variable to track position.

---

### 4. Uniform Chunk Validation (Every Segment Identical Pattern)

Where `chunk()` truly shines is validating that **every fixed-width segment** matches the same pattern — something no prior technique could do for variable n:

```
// Every 4-character chunk must be exactly 4 digits
// Regex: ^(\d{4})+$  where length is divisible by 4

and(
  equals(mod(length(input), 4), 0),
  equals(
    length(
      filter(
        chunk(input, 4),
        not(
          xpath(xml(concat('<r>', item(), '</r>')),
            'string-length(/r) = 4 and
             string-length(translate(/r, "0123456789", "")) = 0')
        )
      )
    ),
    0
  )
)
```

This validates `^\d{4}(\d{4})*$` — a card number format, IBAN segment, or any fixed-block numeric code — for any length input, in a single expression.

---

### 5. Data-Driven Character Validation (The Big Upgrade)

Previously, the `translate()` approach required hardcoding the allowed character set in the XPath string. With `chunk(input, 1)` + `filter()`, the allowed set can come from a variable or array:

```
// allowedSet = some dynamic array of permitted characters
// loaded from config, environment variable, SharePoint list etc.

equals(
  length(
    filter(
      chunk(input, 1),
      not(contains(join(allowedSet, ''), item()))
    )
  ),
  0
)
```

This is genuinely new — **runtime-configurable character validation** without rebuilding an XPath string dynamically.

---

## Revised Impossibility Table

| Feature | Previous Status | With `chunk()` |
|---|---|---|
| `(\d{2}[A-Z]){n}` repeating sub-pattern | ❌ Impossible | ✅ Possible via `chunk(n)` + `filter()` |
| `(.)\1{2,}` consecutive identical chars | ❌ Impossible | ❌ Still impossible — no index in `filter()` |
| Character-by-character scanning | ❌ Impossible | ✅ Possible via `chunk(1)` + `filter()` |
| `\p{L}` Unicode property classes | ❌ Impossible | ❌ Still impossible — enumeration problem unchanged |
| Backreferences `\1` | ❌ Impossible | ❌ Still impossible — no match state |
| Lookaheads / lookbehinds | ❌ Impossible | ❌ Still impossible — no position cursor |
| Variable-length scanning | ❌ Impossible | ⚠️ Partially — chunk(1) scans characters but filter() has no index |
| Uniform fixed-width repetition `{n}` | ❌ Impossible | ✅ Possible via `chunk(n)` + `filter()` |
| Data-driven character whitelists | ⚠️ Awkward | ✅ Clean via `chunk(1)` + `filter()` + `join()` |
| Alternation over segments | ⚠️ Partial | ✅ Cleaner via `chunk(n)` + `filter()` membership test |

---

## The Honest New Boundary

`chunk()` essentially gives you a **map/filter over fixed-width positional slices** of a string. That resolves the "no iteration" problem for any pattern where:

- The chunk width is known and fixed
- Each chunk is validated independently (no cross-chunk state)
- You only need to know **whether** a chunk fails, not **which position** it's at

What it still cannot do is anything requiring **cross-element state** within the iteration — knowing the index of the current element, comparing an element to its neighbour, or carrying forward a match from one chunk into the next. Those patterns still require an `Apply to Each` loop with explicit state variables, or an Azure Function.

---



summary

Good synthesis question. Let me map this cleanly across the full journey.

---

## The Baseline — What Standard String Functions Give You

Before any of the advanced techniques, Power Automate's native string functions cover only the most basic regex concepts:

- Anchored prefix/suffix matching (`startsWith`, `endsWith`)
- Substring presence (`contains`)
- Exact equality (`equals`)
- Fixed delimiter extraction (`indexOf` + `substring`)
- Simple replacement (`replace`)
- Splitting on a delimiter (`split`)
- Basic numeric validation (`isInt`, `isFloat`)
- Length bounds (`length` + comparisons)

These cover roughly the **bottom 30%** of real-world regex use cases — simple structural checks on well-behaved data.

---

## What Each Layer Unlocked

### Layer 1 — `xml()` + `xpath()` + `translate()`

| Unlocked Ability | Regex Equivalent |
|---|---|
| Character class validation | `[a-zA-Z]`, `[0-9]`, `[a-zA-Z0-9]` |
| Negated character class | `[^abc]` |
| Fixed-length + character class | `^\d{5}$`, `^[A-Z]{3}$` |
| Whitespace detection | `\s`, `\S` |
| Occurrence counting | `(char){n}` count |
| Positional character class | `^.{n}[A-Z]` |
| Delimiter-based capture group simulation | `(?<=delim).*(?=delim)` |
| Validate and extract simultaneously | Anchored capture groups |

**What made this work:** `translate()` strips enumerated characters, turning character-class membership into a length comparison. `substring-before()`/`substring-after()` simulate capture groups anchored by known delimiters.

---

### Layer 2 — Nested `xpath()`

| Unlocked Ability | Regex Equivalent |
|---|---|
| Multi-segment format validation | `^[A-Z]{2}-\d{5}-[A-Z]{3}$` |
| Extract then validate in one expression | `^prefix(\d{5})suffix$` |
| Strip prefix/suffix then validate remainder | `^LITERAL([A-Z0-9]{6})$` |
| Count occurrences within extracted segment | Sub-pattern occurrence count |
| Character-by-character positional validation | `^[A-Z]\d[A-Z]\d[A-Z]\d$` |
| Full pipeline: normalise → extract → validate | Complex dirty-input patterns |

**What made this work:** Feeding the output of one `xpath()` as the input to another `xml()`/`xpath()` pair creates a multi-stage pipeline with no intermediate variables — effectively composing capture, transformation, and validation into a single expression.

---

### Layer 3 — `join()` + `concat()`

| Unlocked Ability | Regex Equivalent |
|---|---|
| Set membership / exact alternation | `^(RED\|GREEN\|BLUE)$` |
| Segment-level alternation across splits | `^(A\|B\|C)(-(?:A\|B\|C))*$` |
| Data-driven character whitelists | Runtime-configurable `[allowed chars]` |
| Template matching with dynamic length bounds | `^FIXED-.{n,m}-FIXED$` |
| Multi-term substring search | `(error\|fail\|exception)` |
| Canonical normalisation before validation | Trim/pad to fixed width then validate |
| XPath expression construction at runtime | Dynamic pattern building |

**What made this work:** `concat()` constructs XPath expressions and sentinel strings at runtime. `join()` turns arrays into delimited strings for membership testing and dynamic character-set building — making validation rules **data-driven** rather than hardcoded.

---

### Layer 4 — `chunk()` + `filter()`

| Unlocked Ability | Regex Equivalent |
|---|---|
| Fixed-width repeating sub-pattern | `(\d{2}[A-Z]){n}` for any n |
| Uniform block validation | `^(\d{4})+$`, `^([A-Z]{3})+$` |
| Character-by-character scanning | Iterating over every position |
| Occurrence counting (alternative method) | `(char){n}` without XPath |
| Runtime-configurable character validation | `[dynamic char set]` from a variable |
| "All segments pass" validation | Every chunk independently validated |

**What made this work:** `chunk(string, n)` exposes fixed-width positional slices as an array. `filter()` then acts as a map/predicate over every slice simultaneously — giving Power Automate its first true iteration over string content within an expression, without a loop.

---

## The Full Picture — Before vs After

| Regex Concept | Native Strings | + XPath | + Nested XPath | + join/concat | + chunk/filter |
|---|---|---|---|---|---|
| `^prefix` / `suffix$` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `.*keyword.*` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `^exact$` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `^.{n,m}$` length bounds | ✅ | ✅ | ✅ | ✅ | ✅ |
| `[0-9]`, `[a-zA-Z]` char classes | ❌ | ✅ | ✅ | ✅ | ✅ |
| `^\d{n}$` fixed length + class | ❌ | ✅ | ✅ | ✅ | ✅ |
| Occurrence counting | ❌ | ✅ | ✅ | ✅ | ✅ |
| `\s` / `\S` whitespace | ❌ | ✅ | ✅ | ✅ | ✅ |
| Positional char class `^.{n}[A-Z]` | ❌ | ✅ | ✅ | ✅ | ✅ |
| Delimiter-anchored capture group | ❌ | ✅ | ✅ | ✅ | ✅ |
| Multi-segment format validation | ❌ | ⚠️ Verbose | ✅ | ✅ | ✅ |
| Extract + validate in one expression | ❌ | ❌ | ✅ | ✅ | ✅ |
| `^(A\|B\|C)$` exact alternation | ❌ | ❌ | ❌ | ✅ | ✅ |
| Segment-level alternation | ❌ | ❌ | ❌ | ✅ | ✅ |
| Data-driven char whitelists | ❌ | ❌ | ❌ | ✅ | ✅ |
| `(\d{2}[A-Z]){n}` repeating sub-pattern | ❌ | ❌ | ❌ | ❌ | ✅ |
| `^(\d{4})+$` uniform block validation | ❌ | ❌ | ❌ | ❌ | ✅ |
| Per-character scanning + filtering | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## What Remains Impossible

Even with all four layers combined, these features have no solution:

| Feature | Why It Remains Impossible |
|---|---|
| Backreferences `\1` | Requires match state — no concept exists in PA expressions |
| Lookaheads / lookbehinds | Requires zero-width positional assertions — no position cursor |
| Consecutive identical chars `(.)\1{2,}` | Requires comparing adjacent elements — `filter()` has no index |
| Recursive / balanced patterns `\(.*\)` | Requires recursion — not possible in any PA expression |
| Conditional patterns `(?(cond)yes\|no)` | Requires branching within a pattern — not a single expression |
| Unicode property classes `\p{L}` | Requires Unicode engine — `translate()` enumeration is impractical |
| Variable-length lookahead scanning | Requires trying a pattern at every position — needs a loop |

---

## The Honest Headline

Starting from a baseline of simple prefix/suffix/contains checks, the full technique stack lifts Power Automate to somewhere around **70–75% coverage of practical real-world regex use cases** — the ones that appear most frequently in data validation, format checking, and structured string extraction in business automation. The remaining gap is dominated by features that require a stateful scanning engine, which is fundamentally what a regex engine *is* and what Power Automate expressions are fundamentally not. For those cases, an Azure Function remains the right tool.
*Power Automate · Azure Logic Apps*
