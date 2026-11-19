# AI-Friendly Architecture — versão curta

**Slides online:** https://sschonss.github.io/talk-ai-friendly-architecture-short/  
**PDF:** https://sschonss.github.io/talk-ai-friendly-architecture-short/ai-friendly-architecture-short.pdf

Versão curta (~20 min, 25 slides) do deck
[AI-Friendly Architecture](https://sschonss.github.io/talk-ai-friendly-architecture/).
Baseada na série
[AI-Friendly Architecture](https://luizschons.com/series/ai-friendly-architecture/).

A palestra completa está em [sschonss/talk-ai-friendly-architecture](https://github.com/sschonss/talk-ai-friendly-architecture).

- `slides.md` — o deck (25 slides, em 3 atos). Cada slide tem pouco texto; o roteiro está nas **notas do apresentador** (`<!-- ... -->`).
- `themes/aifa.css` — o mesmo tema da palestra completa.
- `assets/` — foto e QR da série.

O deploy no GitHub Pages roda a cada push na `main` (`.github/workflows/pages.yml`).

## Comandos

```bash
npm run dev     # servidor com preview ao vivo
npm run build   # dist/index.html (modo apresentador: tecla P)
npm run pdf     # dist/ai-friendly-architecture-short.pdf
npm run pptx    # dist/ai-friendly-architecture-short.pptx
```
