# TLDR

Info regarding setting up Airtable

## Table of Contents

1. [Shout outs](#shout-outs)
2. [High Level Airtable Setup Steps](#high-level-airtable-setup-steps)
3. [Create a Base in your account](#create-a-base-in-your-account)
4. [Create the following tables in your Base](#create-the-following-tables-in-your-base)
  - [Scripts Table Schema](#scripts-table-schema)
  - [Segments Table Schema](#segments-table-schema)
  - [Miscellaneous Table Schema](#miscellaneous-table-schema)

## Shout outs

This technique was inspired by [Stephen G. Pope](https://www.youtube.com/c/stephengpope)

## High Level Airtable Setup Steps

1. Create an Airtable account
1. Create a Base in your account
1. Create tables in your Base: `Scripts`, `Segments`, `Miscellaneous`
    - check out the `diagrams/airtable_data_hierarchy.png` file for reference

## Create a Base in your account

Called `AAIVA`

## Create the following tables in your Base

### `Scripts` Table Schema

Create a table from scratch and delete all the columns (except the primary field)

- ID (Autonumber) [Primary field]
- Script (Long text)
- Spoken Audio (Attachment)
- Spoken Audio URL (URL)
- Active (Checkbox)

And that's good for now

### `Segments` Table Schema

Create a blank table from scratch and delete all the columns (except the primary field)

- ID (Autonumber) [Primary field]
- Script Segment (Single line text)
- Start Time (Duration)
    - Precision: Hours or smaller
    - Duration Format: h:mm:ss.ss
    - Thousands and decimal separators: Local
- End Time (Duration)
    - Precision: Hours or smaller
    - Duration Format: h:mm:ss.ss
    - Thousands and decimal separators: Local
- Image (Attachment)
- Image URL (URL)
- Image Prompt (Long text)
- Video (Attachment)
- Video URL (URL)
- Video Prompt (Long text)
- Full Script
    - Link to another record
    - Scripts
    - Field name: `Full Script`
    - Disable `Allow linking to multiple records`
    - Lookup field: ID
- Make Video (Checkbox)
- Video Render Request ID (Single line text)

And finally one more little table for fun

### `Miscellaneous` Table Schema

Create a blank table from scratch and delete all the columns (except the primary field)

- ID (Autonumber) [Primary field]
- Name (Single line text)
- Images (Attachment)
- Init Image ID (Single line text)
