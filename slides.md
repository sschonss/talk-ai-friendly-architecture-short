---
marp: true
theme: aifa
paginate: true
lang: pt-BR
title: AI-Friendly Architecture
description: Preparando sistemas reais para agentes de IA
author: Luiz Schons
footer: 'AI-Friendly Architecture · luizschons.com'

---
<!-- _class: cover -->
<!-- _paginate: false -->
<!-- _footer: '' -->

# AI-Friendly Architecture

Preparando sistemas reais para agentes de IA

<br>

**Luiz Schons**

<!--
Apresentação baseada na série de 9 artigos em luizschons.com/series/ai-friendly-architecture.
Transição: começar direto pela pergunta. A apresentação pessoal vem logo depois do gancho.
-->

---

<!-- _class: statement -->

> Seu software está pronto para ser **entendido por uma IA**?

<!--
Abrir com a pergunta. Deixar a plateia pensar alguns segundos.
-->

---

<!-- _class: statement -->

> A IA é a **pessoa nova** do time.

<!--
A analogia que guia a palestra inteira.
- Ela não conhece o domínio, os serviços, as decisões passadas nem onde ficam as respostas.
- Com conhecimento bem documentado, encontra respostas sozinha e ganha autonomia.
- Quando depende de uma pessoa específica, de contexto informal ou da memória de alguém, trabalha pior.
Guardar essa frase: vamos voltar a ela no fim.
Transição: "Antes de seguir, deixa eu me apresentar." 
-->

---

<div class="about">
<div>

# Luiz Schons

<p class="role">Senior Software Engineer @ <b>PicPay</b></p>

<div class="chips">
<span>Guarapuava, Paraná, Brasil</span>
<span>DevParaná</span>
<span>UTFPR</span>
<span>PHP</span>
</div>

<p class="site">luizschons.com</p>

</div>
<img class="photo" src="assets/luiz.jpg" alt="Luiz Schons">
</div>

<!--
Apresentação rápida (30 segundos): quem sou, onde trabalho, de onde venho.
Transição: "Vou dividir a conversa em três atos." 
-->

---

## O caminho

<div class="cols3">
<div class="card"><p class="muted small">Ato 1</p><h3>O problema</h3></div>
<div class="card"><p class="muted small">Ato 2</p><h3>Conhecimento e limites</h3></div>
<div class="card"><p class="muted small">Ato 3</p><h3>Na prática</h3></div>
</div>



<!--
Três atos, construídos a partir da série de 9 artigos.
- Ato 1: o problema. O contexto que o agente precisa não está no código: está em docs, decisões e observabilidade.
- Ato 2: como organizar esse conhecimento (skills por contexto) e como colocar limites que não dependem do prompt (guardrails e fitness functions).
- Ato 3: tudo junto, numa task real, do épico até produção.
Um único exemplo atravessa a palestra: o domínio de pagamentos, o Payment Service e uma feature de reembolso parcial.
-->

---

<!-- _class: divider -->

<div class="kicker">Ato 1 · O problema</div>

# O contexto não está no código

<!--
Primeiro ato: por que agentes erram em sistemas reais, e por que o problema raramente é só o modelo.
-->

---

## O problema nem sempre é o modelo

<div class="compare">
<div class="compare-row">
<div class="compare-in"><span>Troca o modelo</span><span>Troca o prompt</span></div>
<span class="compare-arrow">→</span>
<div class="compare-ctx bad">Contexto ruim</div>
<span class="compare-arrow">→</span>
<div class="compare-out bad">Decisão fraca</div>
</div>
<div class="compare-row">
<div class="compare-in"><span>Mesmo modelo</span><span>Mesmo prompt</span></div>
<span class="compare-arrow">→</span>
<div class="compare-ctx good">Contexto bom</div>
<span class="compare-arrow">→</span>
<div class="compare-out good">Decisão assertiva</div>
</div>
</div>

<p class="muted">Muitas vezes, o problema é o <b>sistema</b>.</p>

<!--
Leitura do desenho: na linha de cima, trocamos o modelo e reescrevemos o prompt, mas o contexto continua ruim, e a decisão continua fraca. Na de baixo, mantemos o mesmo modelo e o mesmo prompt, só melhoramos o contexto, e a decisão fica assertiva. O gargalo está no sistema de contexto, não no modelo.

Como chegamos aqui:
1. Aprendemos a escalar sistemas.
2. Aprendemos a modularizar código.
3. Separamos responsabilidades, distribuímos serviços, organizamos times por contexto.
4. Agora: construir software que agentes de IA também entendam, não só pessoas.

A explicação comum para erros de IA é "o modelo não é bom o suficiente". Às vezes é verdade.
Mas sistemas são desenhados para quem JÁ tem contexto. Um dev da casa sabe:
- onde está a documentação relevante;
- quais decisões estão em ADRs antigas;
- qual serviço é dono de cada responsabilidade;
- onde olhar num incidente;
- quais métricas indicam problema real.
O agente não sabe nada disso por padrão. Contexto espalhado, escondido ou inconsistente derruba a qualidade da resposta.
-->

---

## O que o código mostra

<div class="cols">
<div class="card good">

### Mostra
- Implementação
- Estrutura
- O que existe hoje

</div>
<div class="card bad">

### Não mostra
- Por quê
- Trade-offs
- O que foi descartado
- Produção

</div>
</div>

<!--
Armadilha: "a IA lê código, então entende o sistema".
O código não mostra: por que a decisão foi tomada, qual problema de negócio resolve, quais trade-offs foram aceitos, quais partes são sensíveis, o que foi tentado e descartado, como se comporta em produção.
Um sistema maduro vive além do repo: docs, observabilidade, incidentes, tickets, ADRs, dashboards, memória do time.
-->

---

## Incidente: latência no checkout

<p class="muted small">Sem contexto</p>
<div class="flow muted-arrows">
<div class="bad">Docs erradas</div><span>→</span>
<div class="bad">Logs errados</div><span>→</span>
<div class="bad">Hipótese genérica</div>
</div>

<br>

<p class="muted small">Com contexto</p>
<div class="flow">
<div><code>docs/payments</code></div><span>→</span>
<div>Runbook</div><span>→</span>
<div>Queries</div><span>→</span>
<div class="ok">Hipótese focada</div>
</div>

<!--
Apresentar o domínio que vai nos acompanhar até o fim: pagamentos. Cenário: a latência do checkout começa a subir (vamos chamar de INC-2291).
SEM arquitetura AI-friendly, o agente depende de sorte: procura logs no serviço errado, lê dashboards sem saber quais métricas importam, perde tempo e sugere hipóteses genéricas ("aumente os recursos").
COM contexto organizado o caminho é direto: entra na pasta de docs do domínio, acha o runbook de latência, usa as queries prontas e chega numa hipótese focada.
Em detalhe:
1. Identifica o serviço afetado (Payment Service)
2. Consulta a doc do domínio de pagamentos
3. Correlaciona sinais de observabilidade (latência do provedor, timeouts, deploys)
4. Encontra decisões de arquitetura relacionadas (ADRs)
5. Foca nas hipóteses mais prováveis
Não é mágica: o sistema foi preparado para a IA trabalhar com menos atrito.
Onde começar: incidentes, onboarding e mudanças em áreas críticas.
-->

---

<!-- _class: statement -->

> `200 OK` não significa que o **negócio** está funcionando.

<!--
Erro comum: olhar só CPU, memória, taxa de erro e latência.
- Uma resposta de sucesso pode esconder uma etapa de negócio falhando.
- Poucos erros podem atingir clientes-chave.
- Uma fila lenta pode destruir a experiência.
Conecte eventos de negócio: pagamentos aprovados, pedidos concluídos, documentos processados, transações com ajuda manual.
Nem todo alerta técnico é incidente de negócio; nem todo problema de negócio aparece como erro técnico.
Depois do deploy: quais métricas observar, qual comportamento é esperado, como o antigo se compara ao novo. O agente ajuda a escrever E a verificar a mudança.
-->

---

<div class="dash">
<div class="dash-panel">
<div class="dash-head"><span>Status code · checkout</span><span class="dash-legend"><i class="ok"></i>2xx 99,9% <i class="err"></i>5xx 0</span></div>
<div class="dash-kpis"><div><b class="ok">200 OK</b><span>predominante</span></div><div><b class="ok">0</b><span>erros</span></div></div>
<svg viewBox="0 0 720 380" class="dash-chart"><line x1="70" x2="700" y1="20" y2="20" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="98" y2="98" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="175" y2="175" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="252" y2="252" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="330" y2="330" stroke="#eceae4" stroke-width="1.5"/><rect x="73.9" y="92.6" width="18.4" height="237.4" rx="3" fill="#0f9d6b"/><rect x="73.9" y="90.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="100.2" y="84.7" width="18.4" height="245.3" rx="3" fill="#0f9d6b"/><rect x="100.2" y="82.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="126.4" y="61.5" width="18.4" height="268.5" rx="3" fill="#0f9d6b"/><rect x="126.4" y="59.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="152.7" y="67.5" width="18.4" height="262.5" rx="3" fill="#0f9d6b"/><rect x="152.7" y="65.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="178.9" y="51.2" width="18.4" height="278.8" rx="3" fill="#0f9d6b"/><rect x="178.9" y="49.2" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="205.2" y="54.5" width="18.4" height="275.5" rx="3" fill="#0f9d6b"/><rect x="205.2" y="52.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="231.4" y="65.3" width="18.4" height="264.7" rx="3" fill="#0f9d6b"/><rect x="231.4" y="63.3" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="257.7" y="61.1" width="18.4" height="268.9" rx="3" fill="#0f9d6b"/><rect x="257.7" y="59.1" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="283.9" y="82.7" width="18.4" height="247.3" rx="3" fill="#0f9d6b"/><rect x="283.9" y="80.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="310.2" y="84.6" width="18.4" height="245.4" rx="3" fill="#0f9d6b"/><rect x="310.2" y="82.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="336.4" y="106.0" width="18.4" height="224.0" rx="3" fill="#0f9d6b"/><rect x="336.4" y="104.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="362.7" y="117.0" width="18.4" height="213.0" rx="3" fill="#0f9d6b"/><rect x="362.7" y="115.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="388.9" y="118.2" width="18.4" height="211.8" rx="3" fill="#0f9d6b"/><rect x="388.9" y="116.2" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="415.2" y="114.7" width="18.4" height="215.3" rx="3" fill="#0f9d6b"/><rect x="415.2" y="112.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="441.4" y="134.7" width="18.4" height="195.3" rx="3" fill="#0f9d6b"/><rect x="441.4" y="132.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="467.7" y="130.7" width="18.4" height="199.3" rx="3" fill="#0f9d6b"/><rect x="467.7" y="128.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="493.9" y="115.3" width="18.4" height="214.7" rx="3" fill="#0f9d6b"/><rect x="493.9" y="113.3" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="520.2" y="98.6" width="18.4" height="231.4" rx="3" fill="#0f9d6b"/><rect x="520.2" y="96.6" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="546.4" y="96.7" width="18.4" height="233.3" rx="3" fill="#0f9d6b"/><rect x="546.4" y="94.7" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="572.7" y="88.9" width="18.4" height="241.1" rx="3" fill="#0f9d6b"/><rect x="572.7" y="86.9" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="598.9" y="62.5" width="18.4" height="267.5" rx="3" fill="#0f9d6b"/><rect x="598.9" y="60.5" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="625.2" y="75.0" width="18.4" height="255.0" rx="3" fill="#0f9d6b"/><rect x="625.2" y="73.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="651.4" y="47.0" width="18.4" height="283.0" rx="3" fill="#0f9d6b"/><rect x="651.4" y="45.0" width="18.4" height="2" fill="#d6455d" opacity="0"/><rect x="677.7" y="56.9" width="18.4" height="273.1" rx="3" fill="#0f9d6b"/><rect x="677.7" y="54.9" width="18.4" height="2" fill="#d6455d" opacity="0"/><g class="ax"><text x="70" y="364" text-anchor="start">14:00</text><text x="228" y="364" text-anchor="middle">14:15</text><text x="385" y="364" text-anchor="middle">14:30</text><text x="542" y="364" text-anchor="middle">14:45</text><text x="700" y="364" text-anchor="end">15:00</text><text x="56" y="28" text-anchor="end">2k</text><text x="56" y="330" text-anchor="end">0</text></g></svg>
</div>
<div class="dash-panel">
<div class="dash-head"><span>Latência · checkout</span><span class="dash-legend"><i class="p75"></i>p75 <i class="p95"></i>p95 <i class="p99"></i>p99</span></div>
<div class="dash-kpis"><div><b class="warn">1,2 s</b><span>p75</span></div><div><b class="warn">2,2 s</b><span>p95</span></div><div><b class="bad">3,7 s</b><span>p99</span></div></div>
<svg viewBox="0 0 720 380" class="dash-chart"><line x1="70" x2="700" y1="20" y2="20" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="98" y2="98" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="175" y2="175" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="252" y2="252" stroke="#eceae4" stroke-width="1.5"/><line x1="70" x2="700" y1="330" y2="330" stroke="#eceae4" stroke-width="1.5"/>
<line x1="70" x2="700" y1="268" y2="268" stroke="#7a7d86" stroke-width="2" stroke-dasharray="8 8"/>
<text x="78" y="258" class="slo">SLO 800 ms</text>
<polyline points="70.0,316.9 96.2,316.9 122.5,316.5 148.8,315.3 175.0,316.8 201.2,315.9 227.5,315.7 253.8,316.3 280.0,315.9 306.2,317.1 332.5,316.1 358.8,313.6 385.0,309.7 411.2,306.9 437.5,303.1 463.8,298.0 490.0,293.3 516.2,288.2 542.5,281.1 568.8,275.1 595.0,269.5 621.2,261.6 647.5,254.4 673.8,245.8 700.0,238.0" fill="none" stroke="#e8b14a" stroke-width="4" stroke-linejoin="round"/>
<polyline points="70.0,306.0 96.2,303.3 122.5,306.7 148.8,305.5 175.0,304.2 201.2,306.5 227.5,305.2 253.8,307.0 280.0,304.5 306.2,304.2 332.5,303.0 358.8,297.9 385.0,294.7 411.2,286.7 437.5,279.4 463.8,270.9 490.0,261.9 516.2,250.0 542.5,238.4 568.8,228.3 595.0,214.9 621.2,203.9 647.5,187.3 673.8,172.8 700.0,156.0" fill="none" stroke="#c27803" stroke-width="4" stroke-linejoin="round"/>
<polyline points="70.0,287.7 96.2,291.0 122.5,290.4 148.8,288.7 175.0,292.7 201.2,289.9 227.5,291.8 253.8,292.1 280.0,292.4 306.2,285.1 332.5,283.1 358.8,274.2 385.0,263.4 411.2,248.8 437.5,240.7 463.8,223.9 490.0,207.6 516.2,188.5 542.5,170.8 568.8,151.3 595.0,134.6 621.2,112.3 647.5,90.3 673.8,63.6 700.0,38.9" fill="none" stroke="#d6455d" stroke-width="6" stroke-linejoin="round"/>
<g class="ax"><text x="70" y="364" text-anchor="start">14:00</text><text x="228" y="364" text-anchor="middle">14:15</text><text x="385" y="364" text-anchor="middle">14:30</text><text x="542" y="364" text-anchor="middle">14:45</text><text x="700" y="364" text-anchor="end">15:00</text><text x="56" y="28" text-anchor="end">4s</text><text x="56" y="183" text-anchor="end">2s</text><text x="56" y="330" text-anchor="end">0</text></g></svg>
</div>
</div>

<p class="muted small">exemplo ilustrativo</p>

<!--
Exemplo ilustrativo (números fictícios) logo depois da frase "200 OK não significa que o negócio está funcionando".
Esquerda: o painel que todo mundo olha. Quase tudo 200, zero 5xx. Pelo dashboard técnico, está tudo verde.
Direita: a latência do mesmo checkout. O p75 e o p95 sobem, o p99 dispara e passa muito do SLO de 800 ms.
O cliente não vê erro: vê o checkout girando, desiste e o pagamento não acontece. O negócio está quebrado com 200 OK.
Para o agente é igual: se ele só olha taxa de erro, conclui "está saudável". Ele precisa saber quais sinais importam (latência por percentil, pagamentos aprovados) e o que é normal.
-->

---

<!-- _class: divider -->

<div class="kicker">Ato 2 · Conhecimento e limites</div>

# Organizar o conhecimento. Definir limites.

<!--
Segundo ato: se o contexto está espalhado, como organizá-lo para o agente (skills por contexto) e como garantir limites que não dependem do prompt (guardrails e fitness functions).
-->

---

## Uma Context Skill bem delimitada

<div class="cols5">
<div class="card big">Vocabulário</div>
<div class="card big">Contratos</div>
<div class="card big">Fontes</div>
<div class="card big">Limites</div>
<div class="card big">Ownership</div>
</div>

<br>

<p class="muted">10 skills para uma pergunta simples? <b>Fronteira errada.</b></p>

<!--
O que uma Context Skill sabe, com o exemplo de pagamentos:
1. Vocabulário: transação, autorização, captura, reembolso. Ex.: distinguir transação AUTORIZADA de CAPTURADA antes de qualquer hipótese.
2. Contratos: APIs, eventos e schemas; o que exige versionamento.
3. Fontes de verdade: docs, logs, métricas e traces, deploys recentes, queries read-only.
4. Limites: read-only, sem comandos de escrita, nada sensível no output, o que precisa de aprovação, quando parar e escalar. A skill guia, mas não é a única proteção.
5. Ownership: qual serviço faz o quê e quem é dono da skill, quem revisa quando o contexto muda (ex.: payments-team). Puxa o próximo slide.

Fronteira: decompor não é criar centenas de skills minúsculas.
Uma boa fronteira agrupa o que MUDA junto, é USADO junto e tem DONO relacionado.
Teste: se esse contexto muda, quem precisa revisar? Se envolve times muito diferentes, a fronteira está grande demais.
Comece grande (Observabilidade, Delivery, Dados, Pagamentos, Pedidos, Segurança) e divida só quando houver necessidade real.
-->

---

## Se todo mundo é dono, ninguém é

```yaml
name: payments-context
owner: payments-team
review_frequency: quarterly
sources:
  - payment-service
  - payment-documentation
  - payment-dashboards
```

<!--
Se ninguém atualiza a skill quando a API muda, o agente trabalha com conhecimento velho.
O arquivo não resolve a manutenção sozinho, mas torna a responsabilidade visível.
Onde vive: com o código do time, num repo de plataforma ou num catálogo. Deixe claro: dono, quais agentes usam, fontes externas, versionamento, revisão e permissões.
Observabilidade → time de plataforma, sem cópias de dados de produção. Pagamentos → time do domínio, apontando para ADRs, contratos e runbooks.
Transição: "A skill guia. Mas guiar não é garantir."
-->

---

<!-- _class: statement -->

<p class="kicker-dark">Guardrails</p>

> Prompt **orienta**. Não **protege**.

<br>

<p class="muted">A skill guia. O guardrail garante.</p>


<!--
"Use a API de produção só para leitura." "Nunca acesse endpoints de admin."
Regras no prompt não são barreira: o agente recebe docs novas, combina fontes, lê o código; o texto pode ter instruções conflitantes.
Checagens dentro da skill ou do MCP têm a mesma fraqueza.
MCP conecta agentes a sistemas, mas não é um limite de segurança. A autorização vive no serviço que protege o recurso.
A skill GUIA. O guardrail GARANTE.
A skill deve dizer: o que é read-only, o que precisa de aprovação, quais dados podem ser expostos, quando a evidência é insuficiente, quando passar para um humano.
MAS limites não podem existir só como instrução. Guardrails fixos checam permissões, argumentos, ambiente e impacto ANTES da ação.
Nunca assuma que o agente vai seguir uma regra de segurança sozinho.
Como começar: pegue um problema recorrente e observe o expert. Quais perguntas vêm primeiro? Quais fontes? Como interpreta? Que hipóteses descarta? Quando escala? Que output é útil?
Transição para o Ato 3: contexto organizado, limites fora do prompt. Agora vamos ver tudo isso numa task real.
-->

---

<!-- _class: divider -->

<div class="kicker">Ato 3 · Na prática</div>

# Do épico à produção

<!--
Terceiro ato: juntar tudo numa task real do domínio de pagamentos, o reembolso parcial, do épico até produção.
-->

---

<!-- _class: statement -->

> “Quero devolver **só alguns itens** do pedido.”

<!--
Épico de exemplo: reembolso parcial.
Parece pequeno, mas toca: criação de pedido, cálculo de total, dono do pagamento, ligação item ↔ transação, regras de cancelamento, estoque, notificações, dados do provedor.
Um agente trabalhando só a partir do épico produz uma solução plausível, mas errada.
-->

---

## A entrega inteira

<div class="phases">
<div class="phase">
<div class="phase-title">Entender</div>
<div class="phase-step">Questionar</div>
<div class="phase-step">Spike</div>
<div class="phase-step">Decisão</div>
</div>
<div class="phase">
<div class="phase-title">Planejar</div>
<div class="phase-step">Sucesso</div>
<div class="phase-step">Tarefas e testes</div>
<div class="phase-step">Sinais</div>
</div>
<div class="phase hl">
<div class="phase-title">Construir</div>
<div class="phase-step">Código</div>
</div>
<div class="phase">
<div class="phase-title">Operar</div>
<div class="phase-step">Deploy</div>
<div class="phase-step">Monitorar</div>
<div class="phase-step">Docs</div>
</div>
</div>

<p class="muted">O código é uma etapa, não a entrega.</p>

<!--
O agente participa do ciclo inteiro, não só de escrever código.
- Entender: questionar o épico, fazer o spike, registrar a decisão.
- Planejar: definir sucesso, quebrar em tarefas, definir testes, decidir sinais e alertas ANTES do código.
- Construir: o código. Repare: é uma coluna de quatro, e a menor.
- Operar: deploy com aprovação humana, monitorar contra o baseline, atualizar a documentação com o que foi aprendido.
Entender o problema, decisões, critérios de sucesso, ambientes e sinais podem exigir mais raciocínio do que codar.
-->

---

## Na prática: um link, todo o contexto

<svg class="network" viewBox="0 0 1640 620" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Da task à página do domínio e dela a todas as fontes de contexto">
<defs><marker id="arrc" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker></defs>
<line x1="323" y1="300" x2="525" y2="300" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="45" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="147" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="249" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="351" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="453" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<line x1="868" y1="300" x2="1245" y2="555" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arrc)"/>
<rect x="60.0" y="260.0" width="260" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="190" y="311" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">PAY-1432</text>
<rect x="535.0" y="260.0" width="330" height="80" rx="16" fill="#3b5bdb" stroke="#3b5bdb" stroke-width="2"/>
<text x="700" y="311" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#ffffff">Payment Service</text>
<rect x="1255.0" y="10.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="56" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Repositório</text>
<rect x="1255.0" y="112.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="158" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">ADR-012</text>
<rect x="1255.0" y="214.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="260" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Contrato v2</text>
<rect x="1255.0" y="316.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="362" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Runbook</text>
<rect x="1255.0" y="418.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="464" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">Dashboard</text>
<rect x="1255.0" y="520.0" width="270" height="70" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
<text x="1390" y="566" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="600" fill="#1d1f24">INC-2291</text>
<text x="190" y="384" text-anchor="middle" font-family="Inter, sans-serif" font-size="24" font-weight="700" letter-spacing="2" fill="#7a7d86">INPUT</text>
<text x="700" y="384" text-anchor="middle" font-family="Inter, sans-serif" font-size="24" font-weight="700" letter-spacing="2" fill="#7a7d86">PORTA DE ENTRADA</text>
</svg>

<p class="muted">O input foi um link. O contexto veio da <b>arquitetura</b>.</p>

<!--
Esse é o ponto da palestra inteira, num slide:
- Você mandou um link (a task).
- A task apontou para a página do domínio.
- A página apontou para tudo o que o agente precisa: código, decisão, contrato, runbook, dashboard, incidentes.
O agente não precisou de um prompt gigante nem de alguém explicando o sistema. Ele navegou pela Context Architecture.
E ele não lê tudo: segue só os links relevantes para a tarefa (aqui, ADR-012 e o contrato).
-->

---

## Na prática: do plano ao PR

<div class="chat">
<div class="msg agent"><span class="who">Agente</span><div class="skills"><span class="skill">architecture</span><span class="skill">testing</span></div>Plano: reutilizar a transação e rastrear o total reembolsado, como na ADR-012. Testes de concorrência, idempotência e falha após a confirmação do provedor.</div>
<div class="msg agent"><span class="who">Agente</span><div class="skills"><span class="skill">delivery</span></div>PR aberto: <a>#482 · Reembolso parcial por item</a>. Testes passando, fitness functions ok. Rascunho da GMUD pronto.</div>
<div class="msg user"><span class="who">Você</span>Mandei o PR para o time aprovar e estou providenciando a GMUD. Quando estiver tudo aprovado, eu subo e te aviso aqui.</div>
</div>

<!--
- Skills de architecture e testing: o plano se apoia na decisão registrada (ADR-012), não numa ideia nova. Os testes saem dos critérios de sucesso.
- Skill de delivery: o agente PREPARA o PR e o change record. Os guardrails continuam valendo: fitness functions rodam no pipeline (a regra do Ato 2: Payments não importa internos de Orders; o PR passou porque o agente usou o contrato público) e quem aprova é uma pessoa.
- Preparar ≠ executar: o time revisa o PR e a GMUD (gestão de mudança) passa pela aprovação. Só depois disso a pessoa sobe em produção pelo pipeline.
-->

---

## Na prática: deploy e monitoramento

<div class="chat">
<div class="msg user"><span class="who">Você</span>GMUD aprovada, subi em produção. Monitora pra mim?</div>
<div class="msg agent"><span class="who">Agente</span><div class="skills"><span class="skill">observability</span></div>Acompanhando o <a>dashboard de pagamentos</a>. 30 min após o deploy:
<ul><li>Reembolsos parciais confirmados, sem duplicidade</li><li>Latência p95 estável</li><li>Nenhuma divergência entre provedor e base</li></ul>Aviso se a taxa de falha passar de 2%.</div>
<div class="msg agent"><span class="who">Agente</span><div class="skills"><span class="skill done">docs atualizadas</span></div>Atualizei a página do Payment Service com o runbook do reembolso parcial.</div>
</div>

<!--
- Skill de observability: o agente sabe qual dashboard olhar porque o link está na página do domínio. Compara com o baseline e com os critérios de sucesso definidos antes do código.
- O limite de alerta (2% de falha) foi definido pelo time; o agente acompanha e avisa, a decisão de continuar ou reverter é humana.
- Fechando o ciclo: a documentação é atualizada, e a próxima task começa com mais contexto.
FIM DA SIMULAÇÃO e do conteúdo principal: do link da task até produção, com a doc atualizada para a próxima task.
Transição: "O que levar daqui."
-->

---

<!-- _class: divider -->

<div class="kicker">Fechamento</div>

# O que levar daqui

---

## Agentes melhores precisam de

<div class="cols4">
<div class="card big">Contexto<div class="small muted">docs e ADRs</div></div>
<div class="card big">Limites<div class="small muted">guardrails</div></div>
<div class="card big">Evidências<div class="small muted">sinais reais</div></div>
<div class="card big accent">Feedback<div class="small muted">doc que evolui</div></div>
</div>

<br>

<p class="muted">Menos adivinhação. Mais <b>arquitetura</b>.</p>

<!--
Fechamento em uma frase: agentes melhores dependem menos de adivinhar e mais de contexto, limites, evidências e feedback.
- Contexto: docs com dono, decisões registradas (ADRs), página do domínio.
- Limites: guardrails e fitness functions, fora do prompt.
- Evidências: observabilidade que conta a história (200 OK não basta).
- Feedback: cada task devolve conhecimento para a doc, e a próxima começa com mais contexto.
Tudo isso ajuda primeiro as pessoas. O agente é só a pessoa nova que chega todo dia.
Transição: série de artigos e perguntas.
-->

---

<div class="qr-slide">
<div>

<p class="kicker-dark">Esta palestra é baseada na minha série de artigos</p>

# AI-Friendly Architecture

<p class="muted">9 artigos</p>

<p class="site">luizschons.com/series/ai-friendly-architecture</p>

</div>
<figure class="qr">
<img src="assets/qr-serie.svg" alt="QR code da série">
<figcaption>Escaneie para ler a série</figcaption>
</figure>
</div>

<!--
Para se aprofundar: a palestra é um resumo da série de 9 artigos (luizschons.com/series/ai-friendly-architecture). Deixar alguns segundos para quem quiser escanear.
Depois, ir para as perguntas.
-->

---

<!-- _paginate: false -->
<!-- _footer: '' -->

<p class="kicker-dark">Obrigado!</p>

# Perguntas?

<div class="contacts">
<div><span class="contact-label">Site</span><span class="contact-value">luizschons.com</span></div>
<div><span class="contact-label">LinkedIn</span><span class="contact-value">linkedin.com/in/luiz-schons</span></div>
<div><span class="contact-label">GitHub</span><span class="contact-value">github.com/sschonss</span></div>
</div>

<!--
Abrir para perguntas. Deixar este slide na tela durante o Q&A.
Contatos: site, LinkedIn e GitHub.
Nesta versão curta não há QR de feedback: o formulário da palestra longa é de outro evento.
-->
