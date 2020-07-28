Step 1 - In Splunk, create a new macro:   Settings > Advanced Search > Search Macros > New Search Macro
Step 2 - Copy/paste the base64decode (Splunk SPL) into the DEFINITION field; Also enter "arg1" into the ARGUMENTS field (this will receive the passed base64 string)
Step 3 - Save the txt file to %SPLUNK_HOME%/etc/system/lookups



Example:  Identify ASCII behind base64 sender name, typically used to identify email impersonations
  index=<YourIndex> event_type=<EmailEventType> 
      "smtp.rcpt_to{}"="*@<YourDomain>"
      "email.from"="=?UTF-8?B?*"

      | rex field=email.from "=?UTF-8\?B\?(?<senderName>[\w\=+\/]*)\?=.*<(?<the_rest>.*)>"
      | `base64decode(senderName)`
      | where isnotnull(ascii)
      | eval reason="VIP Impersonation (base64)"
      | eval ascii=lower(ascii)
      | eval from="(". ascii. ") ". the_rest
