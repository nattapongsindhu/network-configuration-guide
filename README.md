# Network Config Manual

Minimal single-page web app for documenting an enterprise network configuration manual. The project is designed as an Infrastructure-as-Code style runbook: topology, hardware inventory, VLAN/IP plan, firewall policy, VPN settings, virtualization notes, and copyable CLI snippets live together in one deployable static page.

![Network Config Manual preview](assets/preview.png)

## Status

- Phase 1-5 complete
- Visual style aligned with `it-for-me` and `it-asset-tracker`
- Ready for GitHub and Vercel static deployment

## Tech Stack

- HTML single-file SPA
- Tailwind CSS via CDN
- Mermaid.js for dynamic topology diagrams
- Lucide icons via CDN
- Vanilla JavaScript for dark mode, Mermaid refresh, and click-to-copy

## Features

- Interactive Mermaid topology: ISP -> Router -> Firewall -> Core Switch -> Virtual Servers
- Sticky table of contents on desktop
- Dark and light mode toggle
- Hardware, VLAN, and IP plan tables
- Copy buttons for every configuration snippet
- Maintenance log and rollback checklist
- Semantic HTML structure for search and documentation browsing

## Local Usage

```powershell
cd network-configuration-guide
python -m http.server 8085 --directory .
```

Open:

```text
http://localhost:8085/
```

## Vercel Deployment

This project does not require a build step.

1. Create a GitHub repository named `network-config-manual`.
2. Push `index.html`, `README.md`, and `assets/preview.png`.
3. Import the repository in Vercel.
4. Keep the framework preset as `Other` or static.
5. Deploy from the repository root.

Vercel will serve `index.html` automatically as the main page.

## Updating Network Data

1. Update the hardware table in the `Prerequisites & Hardware Specs` section.
2. Update the Mermaid topology in `#diagramSource` and the `data-mermaid-source` attribute.
3. Update the VLAN/IP plan in the `Core Switch` table.
4. Add new command snippets with this pattern:

```html
<article class="snippet" data-title="Name">
  <div class="snippet-head">
    <span>DEVICE CLI</span>
    <button class="copy-btn" data-copy-target="unique-id" type="button">
      <i data-lucide="copy"></i>Copy
    </button>
  </div>
  <pre id="unique-id"><code>commands here</code></pre>
</article>
```

## Verification Checklist

- Mermaid diagram renders without syntax errors.
- Every Copy button targets an existing snippet.
- Live Copy interaction works in the browser clipboard.
- TOC links scroll to the correct section.
- The diagram container supports horizontal overflow on small screens.
- The page uses semantic `<main>`, `<section>`, `<nav>`, and table markup.
- Back up FortiGate and Ruijie startup configurations before applying real network changes.

## Project Log

- Status: Phase 1-5 complete
- Visual style: clean internal-tool layout inspired by `it-for-me` and `it-asset-tracker`
- Next step: create GitHub repository and connect it to Vercel
