# SSTI_One-liner

```waybackurls target.com | grep '=' | qsreplace "bounty{{5*5}}" | httpx -match-regex 'bounty25' -threads 300 -http-proxy http://127.0.0.1:8080/ ```

This one-liner will help you to find  **SSTI** 


