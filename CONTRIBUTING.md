# Contributing

Contributions are welcome: new tools, corrections, broken links, outdated entries, and formatting fixes.

---

## Ways to contribute

- **New tool:** Open an [issue](https://github.com/Nervi0z/blue-team-tools/issues/new?template=add-tool.md) with the tool name, link, and a note on its specific defensive use case
- **Fix existing entry:** Broken link, outdated information, better description — submit a pull request directly
- **Typos and formatting:** Small fixes are fine as pull requests without an issue first
- **Structural errors:** Open an issue

---

## Submitting a pull request

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/blue-team-tools.git
   ```
3. Create a descriptive branch:
   ```bash
   git checkout -b add-velociraptor-endpoint
   git checkout -b fix-zeek-broken-link
   ```
4. Edit the relevant `.md` file in `tools/`
5. Commit using [Conventional Commits](https://www.conventionalcommits.org/) prefixes:
   ```bash
   git commit -m "feat: add Velociraptor to endpoint section"
   git commit -m "fix: update broken Zeek documentation link"
   git commit -m "docs: correct OpenSSL enc example"
   ```
6. Push your branch:
   ```bash
   git push origin your-branch-name
   ```
7. Open a pull request against `main`. Reference any related issue with `Closes #NUMBER`

---

## Tool entry format

Use this structure consistently for all new entries in `tools/*.md`:

```markdown
## Tool name

- **Description:** What it does, focused on defensive relevance.
- **Blue Team use:**
  - Concrete use case 1
  - Concrete use case 2
- **Website:** [url](https://...)
- **Type:** CLI / GUI / Web service / Framework / Platform
- **Platform:** Linux / Windows / macOS / Web
- **Installation:**
  ```bash
  sudo apt install tool
  ```
- **Usage examples:**
  ```bash
  # Real example with explanatory comment
  tool --flag value
  ```
- **Alternatives:** Tool1, Tool2
- **Notes:** Configuration tips, caveats, Blue Team-specific advice.
```

Quality criteria:

- Descriptions must be concrete and focused on defensive use — no generic filler phrases
- Verify all links before submitting
- Preference for open-source and actively maintained tools
- Real, executable commands — not descriptions of what a command does
- One well-documented entry is worth more than three incomplete ones
