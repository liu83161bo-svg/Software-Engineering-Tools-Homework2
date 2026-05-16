# Tool Contracts — Medical Query Assistant

## Tool 1: `search_patient_record`
- **Permission Level**: Read
- **Allowlist**: Always allowed for authenticated physicians.
- **Parameter Schema**:
    {
      "type": "object",
      "properties": {
        "patient_id": {
          "type": "string",
          "pattern": "^[A-Za-z0-9]{6,12}$",
          "description": "Hospital-assigned patient identifier, 6-12 alphanumeric characters"
        }
      },
      "required": ["patient_id"]
    }
- **Confirmation Required**: No.
- **Error Handling**: If patient not found, return `{"error": "patient_not_found", "patient_id": "..."}`. Timeout after 5 seconds.

## Tool 2: `update_prescription`
- **Permission Level**: Write
- **Allowlist**: Only allowed for attending physicians with verified credentials.
- **Parameter Schema**:
    {
      "type": "object",
      "properties": {
        "patient_id": { "type": "string", "pattern": "^[A-Za-z0-9]{6,12}$" },
        "medication": { "type": "string", "minLength": 3, "maxLength": 50 },
        "dosage": { "type": "string", "pattern": "^\\d+(\\.\\d+)?\\s*(mg|g|ml|IU)$", "description": "e.g., '500 mg', '2.5 ml'" },
        "frequency": { "type": "string", "enum": ["once daily", "twice daily", "three times daily", "as needed"] }
      },
      "required": ["patient_id", "medication", "dosage", "frequency"]
    }
- **Confirmation Required**: Yes. Physician must explicitly confirm the prescription details before execution.
- **Error Handling**: If patient is allergic to medication, block the call and return a warning. Timeout after 10 seconds.

## Tool 3: `schedule_appointment`
- **Permission Level**: Write
- **Allowlist**: Allowed for physicians and authorized administrative staff.
- **Parameter Schema**:
    {
      "type": "object",
      "properties": {
        "patient_id": { "type": "string", "pattern": "^[A-Za-z0-9]{6,12}$" },
        "datetime": { "type": "string", "format": "date-time", "description": "ISO 8601 datetime in UTC" },
        "department": { "type": "string", "enum": ["cardiology", "neurology", "pediatrics", "general"] }
      },
      "required": ["patient_id", "datetime", "department"]
    }
- **Confirmation Required**: Yes.
- **Error Handling**: If datetime is in the past, return error. Timeout after 8 seconds.

## General Policies
- Rate limit: maximum 5 tool calls per minute per user.
- All tool calls are logged with user ID, timestamp, parameters, and result.
- Any call to a tool not in the allowlist is rejected with `{"error": "tool_not_allowed"}`.