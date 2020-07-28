# Use a Splunk Macro to Convert base64 string to ASCII

## Process
1. In Splunk, create a new macro:   Settings > Advanced Search > Search Macros > New Search Macro
2. Copy/paste the base64decode (Splunk SPL) into the DEFINITION field; Also enter "arg1" into the ARGUMENTS field (this will receive the passed base64 string)
3. Save the txt file to %SPLUNK_HOME%/etc/system/lookups

## Usage:
1. Pass base64 string as an arguement to the macro (see below:   `base64decode(sendername)` -- where sendername is the base64 string)
2. Returned is the variable _ascii_  -- this contains the ascii value of the string, returned by the macro

## Example:  Identify ASCII behind base64 sender name, typically used to identify email impersonations

  ```index=_YourIndex_ event_type=_EmailEventType_
      "smtp.rcpt_to{}"="*@_YourDomain_"
      "email.from"="=?UTF-8?B?*"
      | rex field=email.from "=?UTF-8\?B\?(?<senderName>[\w\=+\/]*)\?=.*<(?<the_rest>.*)>"
      | `base64decode(senderName)`
      | where isnotnull(ascii)
      | eval reason="VIP Impersonation (base64)"
      | eval ascii=lower(ascii)
      | eval from="(". ascii. ") ". the_rest
      ```


