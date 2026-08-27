# Broadband Faults Amazon Connect + Sierra POC

A single-page microsite for proving the existing DEV Amazon Connect web-chat entry point before attaching a Sierra sandbox agent as a `CUSTOM_BOT` participant.

## Included

- `index.html`: complete static one-page site
- Existing DEV orchestration pattern:
  - AWS WAF browser challenge
  - `POST v1/chat/orchestration`
  - hosted Amazon Connect chat interface client
  - session cookies
  - `PUT v1/chat/contact-attributes`
- `DigitalOffsite` widget configuration
- `text/plain`, `text/markdown`, and `application/json` advertised as supported content types
- A local rich-card visual demonstrator for the desired UI behaviour
- DEV-only warning and integration status indicator

## Important distinction

The rich card rendered by the page itself is deliberately labelled as a **local visual demonstrator**. It does not prove that Sierra currently sends `application/json` through the managed Amazon Connect connector. The live test needs to establish:

1. Sierra joins the DEV Amazon Connect contact.
2. Sierra can send a reply through Amazon Connect.
3. The connector preserves `ContentType: application/json` for a structured reply.
4. The configured customer chat interface renders that content rather than displaying raw JSON.
5. A customer selection returns into the same contact and reaches Sierra.

## Deploy to GitHub Pages

1. Create a new repository.
2. Copy `index.html` to the repository root.
3. Commit and push it.
4. In repository settings, enable GitHub Pages from the main branch root.
5. Add the deployed GitHub Pages origin to any DEV API Gateway, WAF, widget-domain, CSP, or CORS allowlists required by the existing integration.

## DEV configuration to verify

The current file uses the values found in the supplied existing AWS integration source:

- API Gateway: `https://fer0osb9kg.execute-api.eu-west-2.amazonaws.com/dev/`
- WAF challenge integration host
- Hosted chat-interface client and customer interface
- `DigitalOffsite` widget ID and snippet ID

Confirm these are approved for the DEV proof of concept before publishing.

## Sierra wiring

Configure the DEV Amazon Connect contact flow to invoke the customer-owned bootstrap Lambda near the beginning of the chat. That Lambda should call Sierra `/amazonconnect/begin` with the Amazon Connect contact ID, Sierra JWT, agent embed token, `enableContactCenter: true`, and any approved test memory.

Never place the Sierra JWT, embed token, AWS access keys, customer access token, or other secrets in this static site.

## Rich-content test

The page advertises `application/json`, but transport and rendering still depend on the DEV Connect contact and the configured customer chat interface. Start with a single known payload, such as:

```json
{
  "schema": "demo.hub-light-selector",
  "version": "1.0",
  "title": "What are the lights on your Hub doing?",
  "options": [
    { "id": "solid-white", "label": "Solid white light" },
    { "id": "flashing-green", "label": "Flashing green light" },
    { "id": "red", "label": "Red light" },
    { "id": "no-lights", "label": "No lights" }
  ]
}
```

If the hosted customer interface does not recognise that schema, it will need a matching renderer or the proof should use an Amazon Connect-supported interactive message schema.

## Test checklist

- Page loads over HTTPS.
- AWS widget client loads.
- WAF token can be obtained from the GitHub Pages origin.
- Orchestration call succeeds.
- Amazon Connect contact starts.
- Contact attributes include `journey=broadband-faults-poc` and `environment=DEV`.
- Sierra joins as `CUSTOM_BOT`.
- Plain-text round trip succeeds.
- JSON content-type message is received.
- Rich content renders.
- Click/selection returns into Connect.
- Sierra acknowledges the selected value.
- Optional live-agent transfer succeeds.
