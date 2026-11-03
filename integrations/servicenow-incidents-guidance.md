# Using the ServiceNow incidents integration

Use these tools only for Cisco collaboration meeting-room incidents (Executive Meeting Guardian).

1. Open an incident only when the room assessment is BLOCKED or ATTENTION REQUIRED, a fix could not be
   verified with a fresh device read, or the room reports a Warning/Error diagnostic with no approved fix.
2. Build the failure signature `emg-<room id>-<first 16 hex chars of SHA-256 of the sorted failing check keys joined with "|">`
   and call `findIncidents` with `correlation_id=<signature>^active=true` first.
3. If an active incident exists, call `addIncidentWorkNote` with the new evidence. Otherwise call
   `createIncident` with that signature as `correlation_id`, assignment group "Collaboration Operations".
4. Put verified observations, evidence gaps and the recommended action in the description or work note.
   Never include credentials, tokens or API keys.
5. Do not close or resolve incidents; resolution is left to the operations team.
6. Report the incident number back to the user.
