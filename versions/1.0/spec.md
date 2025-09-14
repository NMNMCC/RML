# Recurrence Markup Language (RML) Specification, Version 1.0

- Version: 1.0
- Date: September 14, 2025
- Status: Final

## 1\. Introduction

Recurrence Markup Language (RML) is a declarative language for defining
recurring tasks, events, or schedules. Its goal is to describe complex
recurrence rules in a structured format that is both easy for humans to read and
write, and precise for machines to parse.

The core design goals of RML are:

- **Readability**: The intent of the rules should be clear and close to natural
  language.
- **Precision**: Eliminate ambiguity in handling dates, times, and time zones.
- **Expressiveness**: Able to describe a wide range of scenarios, from simple
  ("every day") to complex ("the second-to-last business day of every month,
  excluding holidays").
- **Extensibility**: Allows for new features to be added in the future without
  breaking backward compatibility.
- **Globalization**: Includes built-in support for multiple calendars, such as
  the Gregorian and Chinese calendars.

## 2\. Conformance and Keywords

The keywords "**MUST**," "**MUST NOT**," "**REQUIRED**," "**SHALL**," "**SHALL
NOT**," "**SHOULD**," "**SHOULD NOT**," "**RECOMMENDED**," "**MAY**," and
"**OPTIONAL**" in this document are to be interpreted as described in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## 3\. Format Definition

An RML document **MUST** be a JSON object that conforms to
[RFC 8259](https://www.rfc-editor.org/rfc/rfc8259).

All RML documents **MUST** be validated against the JSON Schema provided in
Appendix B of this document.

## 4\. Core Calculation Model

When calculating the occurrences of an event, an RML parser **SHOULD** follow
this logical sequence:

1. **Anchor Point Determination**: The `schedule.from` (`YYYY-MM-DDTHH:MM:SS`)
   is used as the starting time anchor for the calculation. Its time zone
   context is determined by `schedule.timezone`.
2. **Candidate Set Generation**: Based on the `frequency` and `interval` defined
   in the `schedule.repeat` object, an infinite sequence of dates and/or times
   is generated starting from the `from` time point. This sequence represents
   the basic recurrence rule.
3. **Refinement Filtering (Modify)**: The rules defined in the `schedule.modify`
   object are used to filter the candidate set. Only candidate occurrences that
   meet all conditions specified in both `on` (date constraints) and `at` (time
   constraints) are kept.
   - For example, for a monthly frequency, `modify.on.day_of_week: ["mo"]` would
     filter for all Mondays in that month. `position: -1` would further filter
     for only the last Monday.
4. **Exclusion**: The rules defined in the `schedule.exclude` object are used to
   remove specified occurrences from the filtered result set. Exclusion rules
   can be based on a list of specific dates (`dates`) or a pattern (`on`).
5. **Boundary Limitation (Limit)**: The end conditions defined in
   `schedule.limit` are applied to the final sequence of occurrences.
   - If `count` is defined, generation stops after the specified number of
     occurrences is reached.
   - If `until` is defined, all occurrences at or after this time are discarded.
6. **Result Output**: The final sequence of event occurrences, which conforms to
   all rules, is output.

> **Note**: The `from` date itself **MAY** be the first valid occurrence in the
> sequence, provided it fully satisfies all `modify` and `exclude` rules. If the
> `from` date does not satisfy these rules, the calculation begins with the
> first qualifying occurrence after the `from` date.

## 5\. Detailed Field Definitions

### 5.1 Top-Level Object

| Key           | Type   | Required? | Description                                      |
| :------------ | :----- | :-------- | :----------------------------------------------- |
| `description` | String | No        | An optional text description of the RML rule.    |
| `schedule`    | Object | Yes       | The core object containing all scheduling rules. |

### 5.2 `schedule` Object

| Key        | Type   | Required? | Description                                                                                                                                                                                     |
| :--------- | :----- | :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `calendar` | String | No        | Specifies the calendar system to use for the calculation. **MUST** be one of the values defined in the `enum` in Appendix B's Schema (`gregorian`, `chinese`, etc.). Defaults to `"gregorian"`. |
| `from`     | String | Yes       | The starting time anchor for the calculation. **MUST** be an ISO 8601 string in `date-time` format.                                                                                             |
| `timezone` | String | No        | An IANA timezone name, such as `"Asia/Shanghai"`. If provided, all calculations should be based on this timezone. It is recommended to provide this field to avoid ambiguity.                   |
| `repeat`   | Object | Yes       | Defines the base recurrence frequency and interval.                                                                                                                                             |
| `modify`   | Object | No        | Defines filtering and constraint rules to precisely select occurrences from the base frequency.                                                                                                 |
| `limit`    | Object | No        | Defines the end conditions for the recurrence.                                                                                                                                                  |
| `exclude`  | Object | No        | Defines exception dates or rules to be excluded.                                                                                                                                                |

### 5.2.1 `repeat` Object

| Key         | Type    | Required? | Description                                                                                                                                                                      |
| :---------- | :------ | :-------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `frequency` | String  | Yes       | The basic unit of recurrence. **MUST** be one of `["yearly", "monthly", "weekly", "daily", "hourly", "minutely"]`.                                                               |
| `interval`  | Integer | No        | The interval for the recurrence, **MUST** be an integer greater than or equal to 1. Defaults to `1`. For example, `frequency: weekly` and `interval: 2` means "every two weeks." |

### 5.2.2 `modify` Object

Composed of two optional `Object`s: `on` (date constraints) and `at` (time
constraints).

#### `on` Object

| Key            | Type    | Description                                                                                                                                                                                                                                                                      |
| :------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `month`        | Array   | The month(s) of the occurrence. Items can be integers from 1-12 or a string representing a leap month (e.g., `'4L'`).                                                                                                                                                            |
| `day_of_month` | Array   | The day(s) of the month for the occurrence. Items can be positive integers from 1-31 or negative integers from -1 to -31, but **MUST NOT** be 0.                                                                                                                                 |
| `day_of_week`  | Array   | The day(s) of the week for the occurrence. Items **MUST** be one of `["mo", "tu", "we", "th", "fr", "sa", "su"]`.                                                                                                                                                                |
| `day_of_year`  | Array   | The day(s) of the year for the occurrence. Items can be positive integers from 1-366 or negative integers from -1 to -366, but **MUST NOT** be 0.                                                                                                                                |
| `position`     | Integer | When multiple dates match the `on` rules within a `frequency` period, this is used to select the Nth one. For example, for a monthly frequency, the combination of `day_of_week: ["tu"]` and `position: 2` means "the second Tuesday of the month." `-1` indicates the last one. |

#### `at` Object

| Key    | Type  | Required? | Description                                                                                                                   |
| :----- | :---- | :-------- | :---------------------------------------------------------------------------------------------------------------------------- |
| `time` | Array | Yes       | The time(s) of day for the occurrence. Each item in the array **MUST** be an ISO 8601 string in `time` format (`"HH:MM:SS"`). |

### 5.2.3 `limit` Object

**MUST** contain either `count` or `until`, but **MUST NOT** contain both.

| Key     | Type    | Description                                                                                                                                      |
| :------ | :------ | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| `count` | Integer | The total number of occurrences. **MUST** be an integer greater than or equal to 1.                                                              |
| `until` | String  | The end date/time for the recurrence. **MUST** be an ISO 8601 string in `date-time` format. Occurrences at or after this time will be discarded. |

### 5.2.4 `exclude` Object

| Key     | Type   | Description                                                                                                     |
| :------ | :----- | :-------------------------------------------------------------------------------------------------------------- |
| `dates` | Array  | An array of strings in ISO 8601 `date` format (`"YYYY-MM-DD"`) for excluding specific dates.                    |
| `on`    | Object | An object with the same structure and meaning as `modify.on`, used to exclude all dates that match the pattern. |

---

The examples are located in the [example folder](./examples).
