# Featherwand AI — Integration Guide

Overview
--------
This document describes how to integrate Featherwand AI with an Apache JMeter-based workflow to analyze test results, generate test plans, and provide AI-guided recommendations.

Prerequisites
-------------
- Featherwand account and API key.
- Java 11+ (for JMeter) and a working JMeter installation.
- Network access to Featherwand API endpoints.

Configuration
-------------
1. Store your API key securely (environment variable recommended):

```powershell
setx FEATHERWAND_API_KEY "your_api_key_here"
```

2. Add Featherwand-related properties to `user.properties` or a separate `featherwand.properties` file:

```properties
featherwand.api.url=https://api.featherwand.ai/v1
featherwand.api.key=${env.FEATHERWAND_API_KEY}
featherwand.project=your-project-name
```

Integration Patterns
--------------------
1. Post-run analysis (recommended):
   - After a JMeter test finishes, export results as CSV or JTL.
   - Send the results file to Featherwand for analysis (error clustering, anomaly detection, suggestions).

2. Test plan generation:
   - Use Featherwand to generate or augment test plans via prompts describing load goals and scenarios.
   - Convert returned test plan snippets into JMeter `.jmx` elements.

Example: Sending results via cURL
--------------------------------
```bash
curl -X POST "${FEATHERWAND_API_URL:-https://api.featherwand.ai/v1}/analyze" \
  -H "Authorization: Bearer $FEATHERWAND_API_KEY" \
  -F "project=your-project-name" \
  -F "file=@/path/to/results.jtl" \
  -F "type=jmeter-jtl"
```

Example: Java client (basic sketch)
----------------------------------
```java
import java.net.http.*;
import java.net.URI;
import java.nio.file.Path;

HttpRequest request = HttpRequest.newBuilder()
  .uri(URI.create("https://api.featherwand.ai/v1/analyze"))
  .header("Authorization", "Bearer " + System.getenv("FEATHERWAND_API_KEY"))
  .POST(HttpRequest.BodyPublishers.ofFile(Path.of("results.jtl")))
  .build();

HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
```

Embedding into CI
-----------------
- Add a pipeline step that runs JMeter in non-GUI mode, saves results, then uploads to Featherwand for analysis.
- Fail the build or create annotations based on Featherwand's severity score or alerts.

Security & Privacy
------------------
- Avoid committing API keys. Use environment variables or CI secret stores.
- Sanitize or redact sensitive payloads before sending (PII, secrets in headers).

Troubleshooting
---------------
- 401 Unauthorized: verify `FEATHERWAND_API_KEY` and endpoint URL.
- Timeouts: ensure network access, increase HTTP client timeouts.

Notes & Next Steps
------------------
- Consider writing a small JMeter Listener plugin to automatically call Featherwand post-run.
- Optionally add example pipeline scripts (GitHub Actions, Azure Pipelines) to this repo.

Support
-------
For Featherwand API-specific questions, consult Featherwand documentation or contact Featherwand support.
