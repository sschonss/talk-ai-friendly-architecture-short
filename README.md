# AI-Friendly Architecture — versão curta

**Slides (PT):** https://sschonss.github.io/talk-ai-friendly-architecture-short/  
**Slides (EN):** https://sschonss.github.io/talk-ai-friendly-architecture-short/en/  
**PDF (PT):** https://sschonss.github.io/talk-ai-friendly-architecture-short/ai-friendly-architecture-short.pdf  
**PDF (EN):** https://sschonss.github.io/talk-ai-friendly-architecture-short/en/ai-friendly-architecture-short.pdf

Versão curta (~25 min, prática, 23 slides) do deck
[AI-Friendly Architecture](https://sschonss.github.io/talk-ai-friendly-architecture/).
Baseada na série
[AI-Friendly Architecture](https://luizschons.com/series/ai-friendly-architecture/).

A palestra completa está em [sschonss/talk-ai-friendly-architecture](https://github.com/sschonss/talk-ai-friendly-architecture).

- `slides.md` — o deck em português (23 slides, em 3 atos). Cada slide tem pouco texto; o roteiro está nas **notas do apresentador** (`<!-- ... -->`).
- `slides.en.md` — the English translation (same 23 slides).
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
