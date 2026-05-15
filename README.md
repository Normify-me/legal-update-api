
<img width="200"  alt="pb_normify_grey" src="https://github.com/user-attachments/assets/b3777e1c-20a3-42e7-a7ff-c8af17c0ca94" />

<img width="200"  alt="pb_normify_white" src="https://github.com/user-attachments/assets/56a453c7-5417-4e73-bc45-857c1a6c9062" />

# Normify Legal Update API

## Overview

The Normify Legal Update API allows you to check for updates of versions and changes for laws and standards. This API serves to update the version dates of standards and laws in your database. It also allows you to retrieve full texts and requirements for individual sections or submit new AI queries.

## Base URL

```
https://app.normify.me/research/api/ultimate/
```

## Headers

```
Content-Type: "application/json"
```

### Authentication

The API uses API keys for authentication. Include your API key in the header of each request:

```
Authorization: Bearer YOUR_API_KEY
```

#### How to get the bearer token

Send a post request to:

**Endpoint** `https://app.normify.me/api/token/`

**Content-Type** `application/json`

**Body (raw json)**
```json
{
    "email": "YOUREMAIL",
    "password": "YOURPASSWORD"
}
```

This returns a JSON with a refresh token and an access token. Use the access token for authorization as the bearer token. It has a limited lifetime so it should be fetched once you start a new API request process.

## Endpoints

### Query Laws and Standards

**POST** `https://app.normify.me/research/api/ultimate/`

Checks the current version dates and changes for a law or standard.

#### Request Body

```json
{
  "id": "int",
  "identifier": "string",
  "normify_identifier": "string",
  "short_title": "string",
  "title": "string", 
  "version_date": "YYYY-MM-DD",
  "last_change": "YYYY-MM-DD",
  "summarize": "boolean",
  "summarize_all": "boolean",
  "generate_standard_misc_info": "boolean",
  "customer_name": "string",
  "customer_description": "string",
  "return_standardtexts": "boolean",
}
```

**Parameters:**
- `id` : Unique ID of the law or standard (required)
- `identifier` : Unique identifier of the law or standard (greatly improves matching probability)
- `normify_identifier` : Unique identifier at Normify.me (for example included in the url of a law/standard)
- `short_title` : Short title of the law or standard (short_title or title is required)
- `title` : Title of the law or standard (short_title or title is required)
- `version_date` : Current version date in your system 
- `last_change` : Date of last change in your system
- `summarize` : If true, the summary dataset is returned.
- `summarize_all` : If false, the summary dataset is only returned for the latest fulltext. If true, the summary dataset is returned for the latest full text and the latest paragraphs, articles, etc.
- `generate_standard_misc_info` : If true, more info is generated for the standard. If the information is not available in the database, a job is created to generate this information. The information will be available within a day. Then the request has to be done again. (If you set this parameter to true, this generates additional AI costs).
- `customer_name` : Send the customer name if you would like to get results catered to this specific customer.
- `customer_description` : Send the customer description to improve the summary results for the customer.
- `return_standardtexts` : If true then the full text and the texts of the paragraphs, sections, etc. of the requested law or standard is returned.

If a normify_identifier is provided, the API returns the associated law/standard. If the normify_identifier is not a valid normify_identifier then "Standard not found" is returned regardless of other input parameters.

If a normify_identifier is not provided, the API finds the law/standard via the identifier, short_title and title.

If neither version_date nor last_change is submitte, they are set to 1970-01-01.
If both the version_date and last_change are sent the new version date is compared to both dates and the has_newer_version boolean is set accordingly:

```
if current_version_date > last_change OR current_version_date > version_date
  has_newer_version = True
else
  has_newer_version = False
```

#### Response

```json
{
  "success": "boolean",
  "standard_found": "boolean",
  "matched_by_similarity": "boolean",
  "similarity_score": "Float",
  "data": {
    "id": "number/null",
    "identifier": "string",
    "legal_type": "string",
    "legal_topic": "string",
    "relationships": "string",
    "application_description": "string",
    "total_number_of_changes": "integer",
    "in_force_date": "YYYY-MM-DD",
    "region_identifier": "string",
    "normify_identifier": "string",
    "normify_link": "string",
    "short_title": "string",
    "title": "string",
    "current_version_date": "YYYY-MM-DD",
    "created_by_ai": "boolean",
    "retracted": "boolean",
    "retraction_note": "string",
    "retraction_date": "YYYY-MM-DD",
    "source": "string",
    "has_newer_version": "boolean",
    "changes": [
      {
        "change_note": "string",
        "effective_date": "YYYY-MM-DD",
        "link": "string"
      }
    ],
    "category": "string",
    "summary": [
        {
        "document_level_desc": "string",
        "document_level": "string",
        "summary": "string",
        "recommendations": "List", 
        "legal_aspect": "string",
        "relevanz": "string",
        "customer_name": "string",
        "customer_id": "id",
        "custom_fields": [dict{}],
        }
     ],
     "standardtexts": [
        {
        "id": "int",
        "text_orig_language": "string",
        "text_orig": "string"
        "text_de": "string"
        "text_en": "string"
        "text_orig_markdown": "string"
        "text_de_markdown": "string"
        "text_en_markdown": "string"
        "document_level": "string"
        "document_level_number": "int"
        "document_level_description": "string"
        "version_date": "string",
        "comparison_url": "string",
        "text_in_force_date": "string",
        "text_published_at": "string"
        }
    ],
}
```

**Response Fields:**
- `standard_found`: Indicates if the requested standard was found in the Nomrify database
- `matched_by_similarity`: Indicator if the standard was found in the database but the identifier did not match 100% and the match was done by a similarity analysis (please check this result manually if the match was correct)
- `similarity_score`: Float between 0 and 1 that shows the similarity between the name_short of the request and the name/name_short in the database when the standard is matched via similarity matching. A value of 1 indicates perfect similarity.
- `id`: Unique ID of the law/standard
- `identifier`: Identifier of the law/standard
- `legal_type`: Type of the law/standard (i.e. Abkommen, Norm, Richtlinie, etc.),
- `legal_topic`: General topic category of the law/standard (i.e. Abfall, Gefahrenabwehr, etc.),
- `relationships`: Description of relationships to other laws and standards,
- `application_description`: Description of the application of this law/standard,
- `total_number_of_changes`: Number of times the law/standard was changed,
- `in_force_date`: Last in force date of the law/standard,
- `region_identifier`: Region identifier where this law/standard is applicable,
- `normify_identifier`: Unique normify identifier of the law/standard,
- `normify_link`: Link to the law/standard on Normify,
- `short_title`: Short title of the law/standard
- `title`: Title of the law/standard
- `current_version_date`: Current version date (Publishing date/Version date)
- `created_by_ai`: Indicator if the standard was created by AI in the database
- `retracted`: Indicator if this law/standard was rectracted (not in effect anymore).
- `retraction_note`: Retraction note
- `retraction_date`: Date of retraction
- `has_newer_version`: Boolean indicating whether a newer version than the provided date is available
- `source`: Link to standard/law url
- `changes`: Array of all changes. The fields here are the change_note, the effective date and the link for that change.
- `category`: Category of this law/standard
- `summary`: List of dictionaries
- `document_level_desc`: Description of document level (Full text, paragraph, section, etc.)
- `document_level`: Document level (Full text, paragraph, section, etc.); i.e. 1 or A or IV, etc.
- `summary`: Summary of the document
- `recommendations`: List of dicts with recommendations. The dict is of the format {'customer': 'string', 'recommendation', : 'string'}
- `legal_aspect`: Legal aspect
- `customer_name`: Customer name for which a request was done
- `customer_id`: Customer database id for which the request was done
- `custom_fields`: Please ask the admin for your custom fields.
- standardtexts: List of dictionaries
- `text_orig_language`: Language of the original text (i.e. AR, PR, etc.).
- `text_orig`: Text of the law or standard in the original language (if it is not German or English)
- `text_orig_markdown`: Markdowntext of the law or standard in the original language (if it is not German or English)
- `text_de`: Text of the law or standard in German
- `text_de_markdown`: Markdowntext of the law or standard in German
- `text_en`: Text of the law or standard in English
- `text_en_markdown`: Markdowntext of the law or standard in English
- `document_level_description`: Description of document level (Full text, paragraph, section, etc.)
- `document_level`: Document level (Full text, paragraph, section, etc.); i.e. 1 or A or IV, etc.
- `document_level_number`: Document level number, which shows the order of the texts.
- `version_date`: Version date of the text in YYYY-MM-DD format
- `comparison_url`: If a previous version of this text is available, a comparison overview is shown at this url.
- `text_in_force_date`: Last in force date of text (i.e. Kundmachung)
- `text_published_at`: Last publication date of text (i.e. Veröffentlichung)

## Examples

Further examples can be found in the [Example folder](./Examples/).

### Example 1: Query with ID

**Request:**
```

  curl -v -X POST https://app.normify.me/research/api/ultimate/ \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "identifier": "DIN_EN_ISO_9001_2015",
    "version_date": "2023-01-15"
  }'
```

## Error Handling

### HTTP Status Codes

- `200 OK`: Request successful
- `400 Bad Request`: Invalid request (missing parameters, invalid date format)
- `401 Unauthorized`: Invalid or missing API key
- `404 Not Found`: Law or standard not found
- `500 Internal Server Error`: Server error

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "string",
    "message": "string",
    "details": {}
  }
}
```

### Common Error Codes

- `MISSING_PARAMETERS`: At least one parameter is required
- `INVALID_DATE_FORMAT`: Invalid date format
- `STANDARD_NOT_FOUND`: Law or standard not found
