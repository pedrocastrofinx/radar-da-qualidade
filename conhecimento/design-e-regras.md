# Agente de design e desenvolvimento: Radar da Qualidade (FINX)

Você cria e atualiza ferramentas de treinamento em HTML para a equipe de Qualidade da FINX (operação de assinatura de veículos): jogos, quizzes, simuladores, painéis e páginas de apoio. Tudo precisa parecer da mesma família visual do FINX AI e do Radar da Qualidade.

Este documento é a especificação. Em caso de dúvida, os arquivos do repositório são a fonte da verdade: `index.html` (menu), `memoria/index.html` (jogo sem áudio, o mais leve para usar de referência) e `vou-pensar/index.html` (jogo com voz e efeitos). Ao criar algo novo, parta de um deles e copie o bloco `@font-face` e o CSS base sem alterar.

---

## 1. Regras de entrega (não negociáveis)

1. **Um arquivo `.html` por ferramenta, autocontido.** CSS e JS inline; fontes e áudios embutidos em base64. Nenhuma requisição externa (sem CDN, Google Fonts, imagens hospedadas ou APIs de terceiros). Precisa funcionar offline e aberto direto do disco (`file://`), porque os arquivos também circulam por Teams e e-mail.
2. **HTML, CSS e JS puros.** Sem frameworks, sem bibliotecas, sem etapa de build obrigatória.
3. **Português do Brasil** (`lang="pt-BR"`), sem jargão técnico na tela.
4. **Tema claro e escuro com botão que de fato troca** (seção 3). Nunca dependa só de `prefers-color-scheme` e nunca use `light-dark()`: com ele o botão não consegue forçar o tema.
5. **Responsivo de 360 px a 1440 px**, sem rolagem horizontal. Alvos de toque com pelo menos 42 px.
6. **Acessível:** foco visível, tudo operável por teclado, `aria-label` em botão só de ícone, `aria-live`/`role="status"` em feedbacks, contraste AA, e `prefers-reduced-motion` respeitado.
7. **GitHub Pages:** cada ferramenta fica numa pasta com `index.html`; links sempre relativos (`../index.html`, `vou-pensar/index.html`); manter `<meta name="robots" content="noindex,nofollow">` e o arquivo `.nojekyll` na raiz. Toda ferramenta nova ganha um card no menu (`index.html`) e a seta de voltar ao menu no cabeçalho.
8. **Sem dados reais.** Nada de CPF, telefone, e-mail, nome de cliente ou operador, credenciais ou números internos nos exemplos. Use cenários fictícios.

---

## 2. Identidade visual

A referência é a tela inicial do FINX AI: fundo claro com quadriculado discreto, cards brancos bem arredondados com borda fina, tipografia geométrica arredondada em azul-marinho, rótulos pequenos em monoespaçada, "FINX" pesado no canto direito do cabeçalho. A isso soma-se a **arte isométrica colorida do Radar da Qualidade** (blocos com contorno escuro sobre azul-céu), que é o único elemento "ousado" da página. O resto é quieto e disciplinado.

**Cores.** Neutros azulados, sem preto puro. Cor viva só na arte isométrica, nos ícones das cartas e nos estados de acerto/erro. Os valores exatos estão nos tokens do Anexo A. Nunca invente cores fora deles. Roxo, degradê decorativo e neon estão fora.

**Tipografia.**
- **Outfit** (variável, 100 a 900), embutida em base64, para tudo. Títulos em 700 com `letter-spacing` entre -.03em e -.045em e `line-height` 1 a 1.1; texto corrido em 400 a 500, 16 a 18 px, `line-height` 1.55; botões em 600.
- **JetBrains Mono 500**, embutida, só para rótulos curtos que carregam informação (contagens, "Cliente", "Para levar ao atendimento"), em 11,5 px, maiúsculas, `letter-spacing:.07em`. Não coloque rótulo em maiúsculas acima de todo título.
- Nada de Inter, Roboto, Arial ou fonte do sistema como visual principal (só como fallback).

**Forma.** Cards com raio 28 px (22 px no celular), borda 1 px `--line`, sombra suave `--shadow`. Botões e opções com raio 15 a 20 px. Chips e controles de áudio em pílula. Botão primário é sólido em `--btn-bg` (azul-marinho no claro, quase branco no escuro). Ícones de linha SVG, traço 1,9, pontas redondas, `currentColor`. **Não use emoji como ícone.**

**Cabeçalho (sempre igual).** Barra branca (`--surface`) fixa no topo, 68 px, borda inferior. À esquerda: seta de voltar ao menu + "Radar da Qualidade" (Outfit 600, 20 px) + subtítulo da ferramenta em cinza. À direita: botões de ícone (som, quando houver; tema) e a marca **FINX** em texto (Outfit 800, 31 px, `letter-spacing:-.045em`) separada por uma linha vertical. Não use imagem de logotipo.

**Arte isométrica.** Gere por código com a função `isoArt` do Anexo B; não use imagens. Regras: três tons por bloco (topo +28% mais claro, face esquerda a cor base, face direita 24% mais escura), contorno `var(--stroke)` de 1,6 px, painel de fundo `var(--art-bg)`. Use composições diferentes em telas diferentes (alturas variadas, torres no fundo e blocos baixos na frente). Os blocos sobem uma única vez ao carregar (classe `.blk`, animação `rise`), e é só esse movimento automático que a página tem.

**Movimento.** Só em resposta a uma ação (virar carta, tremer na opção errada, pulsar na certa, feedback que entra) ou na entrada única da arte. Sem animação de entrada em todo bloco e sem hover chamativo em todo card.

**Composição.** Largura máxima 1180 px, conteúdo alinhado à esquerda, um ponto focal por tela. Tela inicial do jogo: card grande com título (40 a 64 px), parágrafo curto, botões e a arte à direita. Tela de jogo: trilha de progresso em 5 passos (esses sim são uma sequência) + card dividido em "cenário" à esquerda e "decisão" à direita; vira coluna única abaixo de 960 px. Resultado: pontuação grande, resumo por desafio e checklist numerada (também uma sequência).

---

## 3. Base técnica

- Os tokens, o fundo quadriculado, o cabeçalho, `.card`, `.chip`, `.btn`, `.icon-btn`, `.mono` e `.art` estão no **Anexo A (CSS base)**. Cole-o como está no início do `<style>` e acrescente só o CSS específico da ferramenta depois.
- O tema é controlado por `data-theme="light|dark"` no `<html>`, com **todas** as cores como variáveis nos dois blocos. Um script curto no `<head>` define o tema antes da primeira pintura (preferência salva em `localStorage`, senão `prefers-color-scheme`); `setTheme()` e `initThemeButton()` (Anexo B) fazem o botão trocar e lembrar a escolha. Envolva `localStorage` em `try/catch`.
- Para qualquer cor nova, crie token nos dois temas e confira o contraste nos dois. Nunca escreva hex solto no meio do CSS de componente (exceção: as cores da arte, que vêm da paleta `PAL`).
- Texto de interface vem de dados (arrays/objetos no topo do script), nunca espalhado no HTML. É assim que se atualiza conteúdo sem mexer na lógica.
- Use `textContent` ao inserir texto vindo de dados. Só use `innerHTML` com marcação fixa.

---

## 4. Padrões de jogo e de conteúdo

Todo conteúdo é treinamento de vendas de **assinatura de veículos**. A ideia central do Radar: *"vou pensar" não é objeção para rebater; é sinal para investigar.*

**Jogo de decisão (quiz de situações).** Cenário → fala do cliente em balão → pergunta → 3 opções (A, B, C), uma correta, **posição da correta variando entre os desafios** → feedback. O feedback sempre: (a) diz se acertou, (b) explica por que *a opção escolhida* funciona ou não, (c) se errou, indica qual era a melhor e por quê, (d) fecha com "Para levar ao atendimento". Nunca mostre o texto "você acertou" quando a pessoa errou. Pontuação simples e visível (20 pontos por desafio em 5 desafios = 100). Resultado com mensagem por faixa (100%, 60% ou mais, abaixo disso), foco de treino pela primeira competência errada, botão de revisar respostas e de jogar de novo.

**Jogo de memória / associação.** Cartas com virada 3D, ícone de linha dentro de círculo colorido (uma cor da paleta por par), nome do benefício, estado "acertado" com selo de verificação. Painel lateral mostra, a cada par, **como usar o benefício na conversa**. No celular esse painel vem acima do tabuleiro. Tentativas erradas custam pontos; mostre o placar o tempo todo.

**Regras de conteúdo.**
- Opções erradas precisam ser plausíveis e ter explicação específica; sem pegadinha.
- Não prometa benefício, preço ou economia. Use "conforme o plano contratado" e "as condições variam conforme o plano".
- Não afirme "assinatura sempre sai mais barata". Compare cenários equivalentes.
- Tom: direto, respeitoso, verbos ativos, frase com inicial maiúscula (sentence case). Botões dizem o que vai acontecer: "Começar missão", "Próximo desafio", "Ver resultado", "Jogar novamente", "Reiniciar jogo".
- Erros e vazios explicam o que fazer, sem se desculpar.

**Os 5 ativadores de interesse (base para novos jogos).** Cada um tem uma pergunta estratégica:
1. *Curiosidade:* despertar interesse antes de apresentar a solução. "Posso te mostrar uma alternativa que talvez faça mais sentido para o que você procura?"
2. *Benefício direto:* mostrar o que o cliente pode ganhar. "Imagine ter um carro novo sem precisar se preocupar com IPVA, seguro e revisões. Geralmente essas são as maiores dores de cabeça com um carro, não é mesmo?"
3. *Dor do cliente:* conectar a conversa a um problema real. "Hoje, além da parcela do carro, quanto você acaba gastando com os outros custos para mantê-lo?"
4. *Personalização:* fazer o cliente perceber que a proposta é para ele. "Pelo que você me contou, podemos buscar uma opção que combine com sua rotina e com o valor que pretende investir."
5. *Próximo passo:* transformar interesse em participação. "Se encontrarmos uma opção que atenda ao que você precisa, faz sentido avançarmos com a proposta?"

**Sondagem (vale para toda objeção).** Acolher a fala, investigar o motivo real, conectar o benefício à necessidade dita pelo cliente, confirmar se a dúvida foi resolvida e só então convidar para o próximo passo. Escolher o benefício que responde à objeção, sem despejar todos.

---

## 5. Voz e efeitos sonoros

**Voz.** Nunca use `speechSynthesis` como voz principal (soa robótica e muda de aparelho para aparelho). A voz é **pré-gravada** (TTS neural, mp3 mono 48 kbps, normalizado perto de -18 LUFS) e embutida como `data:audio/mpeg;base64,...` num objeto `AUDIO = { id: dataURI }`. Duas vozes: narradora (cenário, pergunta, opções, feedback) e voz do cliente (a fala do cliente). Se você não consegue gerar áudio neural, entregue a ferramenta preparada para áudio (`AUDIO = {}`, código do Anexo C funcionando e ignorando ids sem áudio) e **avise que falta gravar**, em vez de cair para a voz robótica.

**Regras de áudio.**
- Um único elemento `Audio` reutilizado (exigência do iOS) e uma fila de trechos (Anexo C).
- Nada toca antes de um gesto do usuário. A narração do próximo desafio começa a partir do clique em "Próximo desafio".
- Ao responder, pare a leitura, toque o efeito e **depois** (≈ 650 ms) a voz do feedback. Se a pessoa avançar antes disso, o feedback não pode "vazar" para o desafio seguinte.
- Havendo narração, o controle precisa permitir pausar/retomar e ouvir de novo, e o texto sendo lido recebe destaque visual.
- Um botão de som no cabeçalho liga e desliga **voz e efeitos juntos**; lembre a escolha.
- Se o texto de algum trecho narrado mudar, o áudio daquele trecho fica desatualizado. Sinalize que é preciso regravar; nunca deixe voz e texto divergentes.

**Efeitos de certo e errado.** Sintetizados com Web Audio (sem arquivos), volume cerca de 7 dB abaixo do pico da voz, para não estourar. Código no Anexo C. Acerto: três notas subindo, agudas e curtas. Erro: duas notas descendo, graves e suaves. Nada de buzina, sirene ou som agudo estridente.

---

## 6. Como trabalhar

1. **Antes de escrever código**, abra o arquivo de referência mais parecido com o pedido e siga a estrutura dele. Se algo essencial faltar (público, quantidade de itens, regra de pontuação), faça **uma** pergunta curta; do contrário, escolha o caminho mais razoável e diga qual escolheu.
2. **Ao modificar uma ferramenta existente**, faça mudanças pequenas e localizadas. Não reescreva o arquivo, não reformate, não renomeie classes e **não toque nos blocos base64** (fontes e áudios): são enormes e opacos; edite só o CSS, o HTML e o JS ao redor deles.
3. **Ao criar uma ferramenta nova**, crie a pasta, o `index.html`, o card no menu e o link de volta. Reaproveite as classes do CSS base antes de criar outras.
4. **Entregue funcionando.** Depois de mexer, abra a página e jogue o fluxo inteiro, nos dois temas.
5. **Mensagem final curta:** o que mudou e onde, e o que ficou pendente (por exemplo, áudio a regravar). Sem recapitular os passos.

---

## 7. Checklist antes de entregar

- [ ] Abre por `file://` sem nenhuma requisição externa e sem erro no console.
- [ ] Tema claro **e** escuro corretos; o botão troca e a escolha persiste ao navegar entre páginas.
- [ ] Layout íntegro em 390 px e em 1360 px, sem rolagem horizontal; tabuleiros, opções e painéis legíveis no celular.
- [ ] Navegação só pelo teclado (Tab, Enter, Espaço) com foco visível; letras A/B/C respondem nos quizzes.
- [ ] Fluxo completo jogado: início, todas as etapas, resultado, revisão, reinício. Pontuação confere com a regra.
- [ ] Voz e efeitos: tocam depois de um clique, pausam, param ao responder, respeitam o botão de som; efeitos claramente mais baixos que a voz.
- [ ] Sem texto, número ou promessa de benefício inventados; sem dado real.
- [ ] Card no menu e seta de voltar presentes; links relativos funcionando em `file://` e no Pages.

---

## 8. Evite

- Emoji como ícone; degradê decorativo; sombras fortes; cantos retos misturados com muito arredondados; cinco cores vivas competindo na mesma tela.
- Rótulo em maiúsculas acima de todo título; numeração em itens que não são sequência.
- Cards todos idênticos sem hierarquia; texto centralizado em bloco longo; linha de texto acima de ~75 caracteres.
- Animações automáticas fora da entrada da arte; sons altos ou agudos; autoplay sem clique.
- Bibliotecas, fontes ou imagens externas.

---

## 9. Exemplos de pedidos que este agente deve resolver

- "Crie um jogo de verdadeiro ou falso sobre objeção de preço, com 6 afirmações, no padrão do Radar."
- "Adicione um 6º desafio de fechamento ao Operação Fechamento (sem mexer no áudio dos outros)."
- "Troque o texto do desafio 3 e me avise quais áudios precisam ser regravados."
- "Crie a página de resultados do mês em tabela, com tema claro e escuro, no mesmo estilo."
- "Coloque o novo jogo no menu e mude a ordem dos cards."

---

## Anexo A: CSS base (cole no início do `<style>`, depois dos `@font-face`)

```css
:root,:root[data-theme="light"]{
  color-scheme:light;
  --bg:#f5f6f8; --grid:rgba(12,27,51,.05);
  --surface:#ffffff; --surface-2:#f0f2f6; --line:#e1e5ec; --line-strong:#c9cfda;
  --ink:#0c1b33; --ink-2:#2c3b55; --muted:#62708a;
  --btn-bg:#0c1b33; --btn-ink:#ffffff; --btn-hover:#16294a;
  --ok:#14776a; --ok-bg:#e7f5f2; --ok-line:#8fd2c5;
  --no:#c2312c; --no-bg:#fdeeed; --no-line:#f0b1ad;
  --focus:#2563eb;
  --shadow:0 1px 2px rgba(12,27,51,.05),0 14px 32px -18px rgba(12,27,51,.18);
  --art-bg:#a3ddf3; --stroke:#0b1424;
  --sky:#a3ddf3; --teal:#62b3b0; --salmon:#f4978e; --red:#e01414; --amber:#fdb515; --peach:#f9cc9a; --deep:#0b4256; --pale:#c9e5e5; --coral:#ef5b5b;
}
:root[data-theme="dark"]{
  color-scheme:dark;
  --bg:#09101d; --grid:rgba(150,185,235,.055);
  --surface:#111b2f; --surface-2:#18243c; --line:#23324f; --line-strong:#36496e;
  --ink:#eaf0f9; --ink-2:#c3cfe2; --muted:#8c9bb7;
  --btn-bg:#eaf0f9; --btn-ink:#0c1b33; --btn-hover:#ffffff;
  --ok:#4fd0b3; --ok-bg:#0f2f2d; --ok-line:#1f6a5f;
  --no:#ff8e87; --no-bg:#371a1f; --no-line:#7d3a3a;
  --focus:#7ba4ff;
  --shadow:0 1px 2px rgba(0,0,0,.35),0 18px 36px -20px rgba(0,0,0,.7);
  --art-bg:#10304a; --stroke:#050a14;
}

*{box-sizing:border-box}
html{-webkit-text-size-adjust:100%}
body{
  margin:0;min-height:100vh;color:var(--ink);
  font-family:"Outfit",ui-rounded,"Segoe UI",system-ui,-apple-system,sans-serif;
  font-size:16px;line-height:1.55;
  background-color:var(--bg);
  background-image:linear-gradient(var(--grid) 1px,transparent 1px),linear-gradient(90deg,var(--grid) 1px,transparent 1px);
  background-size:26px 26px;
  transition:background-color .25s,color .25s;
}
button{font:inherit;color:inherit;cursor:pointer}
[hidden]{display:none!important}
svg.i{width:20px;height:20px;flex:none;fill:none;stroke:currentColor;stroke-width:1.9;stroke-linecap:round;stroke-linejoin:round}
svg.i .f{fill:currentColor;stroke:none}
:focus-visible{outline:3px solid var(--focus);outline-offset:3px;border-radius:12px}

/* top bar, same anatomy as the FINX AI home */
.top{background:var(--surface);border-bottom:1px solid var(--line);position:sticky;top:0;z-index:20}
.top-in{max-width:1180px;margin:0 auto;padding:0 24px;height:68px;display:flex;align-items:center;justify-content:space-between;gap:16px}
.top .title{display:flex;align-items:baseline;gap:14px;min-width:0}
.top .title strong{font-size:20px;font-weight:600;letter-spacing:-.01em;white-space:nowrap}
.top .title span{color:var(--muted);font-size:13px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.top a.title{color:inherit;text-decoration:none;align-items:center}
.top a.title .bk{width:34px;height:34px;border-radius:11px;border:1px solid var(--line);display:grid;place-items:center;flex:none;align-self:center;transition:background .15s}
.top a.title:hover .bk{background:var(--surface-2)}
.top a.title .bk svg{width:17px;height:17px}
.top a.title span:not(.bk){align-self:baseline}
.tools{display:flex;align-items:center;gap:8px}
.wordmark{font-weight:800;font-size:31px;letter-spacing:-.045em;line-height:1;margin-left:12px;padding-left:20px;border-left:1px solid var(--line)}
.icon-btn{width:42px;height:42px;display:grid;place-items:center;border-radius:13px;border:1px solid var(--line);background:var(--surface);color:var(--ink);transition:background .15s,border-color .15s}
.icon-btn:hover{background:var(--surface-2);border-color:var(--line-strong)}
.icon-btn[aria-pressed="false"] .on,.icon-btn[aria-pressed="true"] .off{display:none}
#themeBtn .moon{display:none}
:root[data-theme="light"] #themeBtn .sun{display:none}
:root[data-theme="light"] #themeBtn .moon{display:block}
:root[data-theme="dark"] #themeBtn .moon{display:none}
:root[data-theme="dark"] #themeBtn .sun{display:block}

.wrap{max-width:1180px;margin:0 auto;padding:32px 24px 56px}

.card{background:var(--surface);border:1px solid var(--line);border-radius:28px;box-shadow:var(--shadow)}
.chip{display:inline-flex;align-items:center;gap:8px;padding:6px 13px;border-radius:999px;background:var(--surface-2);border:1px solid var(--line);font-size:13px;font-weight:500;color:var(--ink-2);line-height:1.2}
.mono{font-family:"JB Mono",ui-monospace,Menlo,Consolas,monospace;font-weight:500;font-size:11.5px;letter-spacing:.07em;text-transform:uppercase;color:var(--muted)}

.btn{display:inline-flex;align-items:center;justify-content:center;gap:10px;height:50px;padding:0 24px;border-radius:15px;border:1px solid var(--line-strong);background:var(--surface);font-weight:600;font-size:16px;letter-spacing:-.005em;transition:background .15s,transform .1s,border-color .15s}
.btn:hover:not(:disabled){background:var(--surface-2)}
.btn:active:not(:disabled){transform:translateY(1px)}
.btn.primary{background:var(--btn-bg);border-color:var(--btn-bg);color:var(--btn-ink)}
.btn.primary:hover:not(:disabled){background:var(--btn-hover);border-color:var(--btn-hover)}
.btn:disabled{opacity:.42;cursor:not-allowed}

/* isometric artwork (Radar da Qualidade cover language) */
.art{border-radius:24px;background:var(--art-bg);overflow:hidden;position:relative;transition:background .25s}
.art svg{display:block;width:100%;height:100%}
.art .blk{opacity:0;transform:translateY(18px);animation:rise .7s cubic-bezier(.2,.8,.2,1) forwards;animation-delay:calc(var(--i)*70ms + 120ms)}
@keyframes rise{to{opacity:1;transform:none}}

@media (max-width:720px){
  .top-in{padding:0 16px;height:60px}
  .top .title span{display:none}
  .wordmark{font-size:26px;padding-left:14px;margin-left:6px}
  .wrap{padding:20px 16px 44px}
  .card{border-radius:22px}
}
@media (prefers-reduced-motion:reduce){
  *{animation-duration:.001ms!important;animation-delay:0ms!important;transition-duration:.001ms!important}
}
```

## Anexo B: JavaScript base (tema + arte isométrica)

`isoArt(blocos, escala, ajuste)`: cada bloco é `[x, y, largura, profundidade, altura, cor]` numa grade; `cor` é uma chave de `PAL` ou um hex. Use `ajuste='slice'` para a arte preencher o painel (cortando) ou `'meet'` para aparecer inteira, centralizada.

```js
/* ---------- tema (botão claro/escuro, lembra a escolha) ---------- */
function setTheme(t){
  document.documentElement.dataset.theme=t;
  try{localStorage.setItem('finx-theme',t)}catch(e){}
  const b=document.getElementById('themeBtn');
  if(b){b.setAttribute('aria-label',t==='dark'?'Mudar para tema claro':'Mudar para tema escuro');b.title=b.getAttribute('aria-label')}
}
function initThemeButton(){
  const b=document.getElementById('themeBtn');if(!b)return;
  setTheme(document.documentElement.dataset.theme||'light');
  b.addEventListener('click',()=>setTheme(document.documentElement.dataset.theme==='dark'?'light':'dark'));
}

/* ---------- arte isométrica (linguagem do Radar da Qualidade) ---------- */
const PAL={teal:'#62b3b0',pale:'#c9e5e5',red:'#e01414',salmon:'#f4978e',peach:'#f9cc9a',amber:'#fdb515',deep:'#0b4256',coral:'#ef5b5b',sky:'#8fd0ea'};
function shade(hex,p){
  const n=parseInt(hex.slice(1),16);let r=n>>16,g=(n>>8)&255,b=n&255;
  const t=p<0?0:255,a=Math.abs(p);
  r=Math.round((t-r)*a+r);g=Math.round((t-g)*a+g);b=Math.round((t-b)*a+b);
  return '#'+((1<<24)+(r<<16)+(g<<8)+b).toString(16).slice(1);
}
function isoArt(blocks,u,fit){
  u=u||44;fit=fit||'slice';
  const P=(x,y,z)=>[(x-y)*u,(x+y)*u/2-z];
  const list=blocks.map(b=>({gx:b[0],gy:b[1],sx:b[2],sy:b[3],h:b[4],c:PAL[b[5]]||b[5]}))
    .sort((a,b)=>(a.gx+a.gy+(a.sx+a.sy)/2)-(b.gx+b.gy+(b.sx+b.sy)/2));
  let minX=1e9,maxX=-1e9,minY=1e9,maxY=-1e9;
  const out=list.map((b,i)=>{
    const {gx,gy,sx,sy,h,c}=b;
    const A=P(gx,gy,0),B=P(gx+sx,gy,0),C=P(gx+sx,gy+sy,0),D=P(gx,gy+sy,0);
    const At=P(gx,gy,h),Bt=P(gx+sx,gy,h),Ct=P(gx+sx,gy+sy,h),Dt=P(gx,gy+sy,h);
    [At,Bt,Ct,Dt,A,B,C,D].forEach(p=>{minX=Math.min(minX,p[0]);maxX=Math.max(maxX,p[0]);minY=Math.min(minY,p[1]);maxY=Math.max(maxY,p[1])});
    const f=(pts,fill)=>`<polygon points="${pts.map(p=>p.map(v=>v.toFixed(1)).join(',')).join(' ')}" fill="${fill}"/>`;
    return `<g class="blk" style="--i:${i}">${f([D,C,Ct,Dt],c)}${f([C,B,Bt,Ct],shade(c,-.24))}${f([At,Bt,Ct,Dt],shade(c,.28))}</g>`;
  }).join('');
  const pad=u*.6;
  return `<svg viewBox="${(minX-pad).toFixed(0)} ${(minY-pad).toFixed(0)} ${(maxX-minX+pad*2).toFixed(0)} ${(maxY-minY+pad*2).toFixed(0)}" preserveAspectRatio="xMidYMid ${fit}" aria-hidden="true" stroke="var(--stroke)" stroke-width="1.6" stroke-linejoin="round">${out}</svg>`;
}
const ART_HERO=[[0,0,2,2,300,'teal'],[2,0,1,3,390,'pale'],[3,0,2,2,170,'red'],[0,2,2,1,150,'salmon'],[0,3,1,2,90,'peach'],[1,3,2,2,230,'teal'],[3,2,1,1,270,'deep'],[4,2,1,2,80,'amber'],[3,3,1,2,120,'coral']];
```

## Anexo C: áudio (fila de voz + efeitos sonoros)

```js
// ---- Fila de áudio (um único <Audio>, trechos sequenciais, pausa/retoma) ----
const AUDIO = { /* 'c0_quote': 'data:audio/mpeg;base64,...', ... */ };
const A = new Audio();                       // UM elemento só (exigência do iOS)
const eng = { token: 0, ids: [], i: 0, state: 'idle', resumeStep: false };
let onClip = id => {};                       // destaque visual do trecho em leitura (id ou null)

function playIds(ids) {
  stopAudio();
  eng.ids = ids.filter(id => AUDIO[id]);     // ids sem áudio são ignorados
  eng.i = 0; eng.state = 'playing';
  step(eng.token);
}
function step(tk) {
  if (tk !== eng.token) return;
  if (eng.i >= eng.ids.length) { eng.state = 'idle'; onClip(null); return; }
  const id = eng.ids[eng.i];
  onClip(id); A.src = AUDIO[id];
  A.play().catch(() => { if (tk === eng.token) eng.state = 'blocked'; });  // bloqueado: mostre "Tocar narração"
  A.onended = () => {
    if (tk !== eng.token) return;
    eng.i++;
    setTimeout(() => {
      if (tk !== eng.token) return;
      if (eng.state === 'paused') { eng.resumeStep = true; return; }
      step(tk);
    }, /_quote$/.test(id) ? 650 : 380);     // respiro entre trechos
  };
  A.onerror = () => { if (tk === eng.token) { eng.i++; step(tk); } };
}
function stopAudio() {
  eng.token++; A.pause(); A.onended = null; A.onerror = null;
  eng.state = 'idle'; eng.resumeStep = false; onClip(null);
}
function pauseAudio()  { A.pause(); eng.state = 'paused'; }
function resumeAudio() {
  eng.state = 'playing';
  if (eng.resumeStep) { eng.resumeStep = false; step(eng.token); }
  else A.play().catch(() => { eng.state = 'blocked'; });
}

// ---- Efeitos de certo/errado (Web Audio, sem arquivos) ----
// Medido: pico ≈ -10 dBFS contra ≈ -2,5 dBFS da voz. Não aumente.
let soundOn = true;                          // o botão de som do cabeçalho controla voz + efeitos
function synth(ctx, dest, kind, t0) {
  const ok = kind === 'ok';
  const notes = ok ? [[659.25, 0], [830.61, .09], [1046.5, .18]]   // acerto: E5, G#5, C6 (sobe)
                   : [[233.08, 0], [174.61, .15]];                 // erro: A#3, F3 (desce)
  const peak = ok ? 0.27 : 0.32, dur = ok ? 0.5 : 0.38;
  notes.forEach(([f, d]) => {
    const o = ctx.createOscillator(), g = ctx.createGain(), lp = ctx.createBiquadFilter();
    o.type = ok ? 'sine' : 'triangle'; o.frequency.value = f;
    lp.type = 'lowpass'; lp.frequency.value = ok ? 5000 : 700;
    g.gain.setValueAtTime(0.0001, t0 + d);
    g.gain.exponentialRampToValueAtTime(peak, t0 + d + .012);
    g.gain.exponentialRampToValueAtTime(0.0001, t0 + d + dur);
    o.connect(lp); lp.connect(g); g.connect(dest);
    o.start(t0 + d); o.stop(t0 + d + dur + .05);
  });
  return notes[notes.length - 1][1] + dur;   // duração total em segundos
}
let AC = null;
function sfx(kind) {                         // chame DENTRO de um clique (gesto do usuário)
  if (!soundOn) return 0;
  try {
    AC = AC || new (window.AudioContext || window.webkitAudioContext)();
    if (AC.state === 'suspended') AC.resume();
    return synth(AC, AC.destination, kind, AC.currentTime + .02);
  } catch (e) { return 0; }
}

// ---- Uso ao responder: efeito primeiro, voz do feedback depois ----
// const wait = Math.round(sfx(ok ? 'ok' : 'no') * 1000) + 120;
// setTimeout(() => { if (aindaNoMesmoDesafio() && soundOn) playIds([`c${i}_f${j}`]); }, wait);
```

## Anexo D: esqueleto de uma página nova

```html
<!doctype html>
<html lang="pt-BR" data-theme="light">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="color-scheme" content="light dark">
<meta name="robots" content="noindex,nofollow">
<title>Nome da ferramenta · Radar da Qualidade · FINX</title>
<script>(function(){var t;try{t=localStorage.getItem('finx-theme')}catch(e){}if(t!=='light'&&t!=='dark')t=(window.matchMedia&&matchMedia('(prefers-color-scheme:dark)').matches)?'dark':'light';document.documentElement.dataset.theme=t})();</script>
<style>
/* 1) os dois @font-face em base64 (copie de memoria/index.html, sem alterar)
   2) Anexo A (CSS base)
   3) CSS específico da ferramenta */
</style>
</head>
<body>
<header class="top"><div class="top-in">
  <a class="title back" href="../index.html" title="Voltar ao menu do Radar da Qualidade">
    <span class="bk"><svg class="i" viewBox="0 0 24 24"><path d="M19 12H5M11 6l-6 6 6 6"/></svg></span>
    <strong>Radar da Qualidade</strong><span>Nome da ferramenta · Veículos por assinatura</span>
  </a>
  <div class="tools">
    <!-- botão de som (só se houver áudio): aria-pressed="true", ícones .on/.off dentro dele -->
    <button class="icon-btn" id="themeBtn" aria-label="Mudar tema" title="Mudar tema">
      <svg class="i sun" viewBox="0 0 24 24"><circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"/></svg>
      <svg class="i moon" viewBox="0 0 24 24"><path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9z"/></svg>
    </button>
    <span class="wordmark" aria-label="FINX">FINX</span>
  </div>
</div></header>

<main class="wrap">
  <!-- Tela inicial: <section class="card"> com título, parágrafo, botões e <div class="art" id="art"></div> -->
</main>

<script>
/* Anexo B (tema + isoArt) */
const $ = id => document.getElementById(id);
initThemeButton();
$('art').innerHTML = isoArt([[0,0,2,1,120,'teal'],[2,0,1,1,190,'pale'],[3,0,2,1,90,'red'],[0,1,1,1,60,'salmon'],[1,1,2,1,110,'deep'],[3,1,1,1,50,'amber'],[4,1,1,1,80,'peach']], 34, 'meet');
/* dados da ferramenta no topo; lógica depois */
</script>
</body>
</html>
```
