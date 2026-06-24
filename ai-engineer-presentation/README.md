# AI Engineer — interview presentation

A short (~13-slide) [Marp](https://marp.app/) presentation telling the "big picture" story for the
**AI Engineer** role at PRIDE Archive (EMBL-EBI): *Towards an Agentic Proteomics Archive*.

The narrative is built around AI systems already shipped into PRIDE production — the
PRIDE chatbot / DocBot (RAG), the RT-AI helpdesk agent, LLM-guided cross-references,
agentic metadata healing, and the BlueSky bot — and where they go next.

## Files

- `pride-ai-engineer-story.md` — the slide deck (Marp markdown, with speaker notes in `<!-- -->`).
- `images/` — figures used by the deck (submission growth, ticket support dashboard, a live cross-reference example).

## Render

```bash
# PDF
npx @marp-team/marp-cli@latest pride-ai-engineer-story.md --pdf --allow-local-files

# or PPTX / HTML
npx @marp-team/marp-cli@latest pride-ai-engineer-story.md --pptx --allow-local-files
```

Or open `pride-ai-engineer-story.md` in VS Code with the **Marp for VS Code** extension for live preview.

## Notes

- Figures and metrics are drawn from PRIDE's 2026 SAB material; **verify all numbers against the
  latest stats before presenting** (e.g. the 8,631 submissions figure is the 2025 full-year value).
- The "Why me" slide has a placeholder bullet to fill in with the candidate's background.
