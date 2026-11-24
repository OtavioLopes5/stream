# Stream de tela sem som — design

## Por quê
Mostrar a tela do PC para uma pessoa pela internet sem que a música dela (Spotify etc.) pare.
No celular, a música só para quando uma página toca áudio — então o site não pode ter som nenhum.

## Objetivos
- Transmitir a tela (ou uma janela) do PC a 30 fps, pela internet.
- Quem assiste abre um link no navegador (celular ou PC), sem instalar nada.
- Zero áudio: a captura não pega som, o vídeo não tem faixa de áudio, o player é sempre mudo e não tem controle de volume.

## Não-objetivos
Áudio, chat, gravação, login, várias salas.

## Anti-objetivo
Qualquer coisa que faça a música de quem assiste parar.

## Abordagens
- **A) Vídeo direto (WebRTC) — escolhida.** O vídeo vai do PC direto para a pessoa. A conexão inicial usa o servidor público grátis do PeerJS. A página é estática e fica na Vercel. Melhor imagem e menos peças.
- **B) Imagens por WebSocket**, com um servidor no PC e um túnel Cloudflare. É o mais garantido para a música, porque não tem player de vídeo, mas roda a uns 5–10 fps. Fica como plano B só para a página de quem assiste, se o iPhone pausar a música.

## Como funciona
- Um arquivo só: `index.html`.
  - **Sem parâmetros: modo transmitir.** O botão "Transmitir" faz o Chrome pedir a tela ou janela (só vídeo, 30 fps). A página mostra o link para copiar e quantas pessoas estão assistindo.
  - **Com `?v=<id>`: modo assistir.** O vídeo é `muted playsinline autoplay`, sem controles, com um botão de tela cheia. Enquanto não houver transmissão, aparece "Aguardando transmissão…" e a página tenta de novo a cada poucos segundos.
- **Conexão:** quem assiste abre uma conexão de dados com o PC, e o PC liga de volta mandando só o vídeo.
- **30 fps:** a captura é pedida a 30 fps, com dica "motion" e preferência por manter o fps. Se a internet apertar, cai a resolução, não o fps.
- **Link fixo:** o ID fica salvo no navegador do PC, então o link é o mesmo de uma transmissão para outra.
- **Rede bloqueada:** se a conexão direta falhar, o PeerJS usa o servidor de repasse grátis dele, que pode ser mais lento.

## Hospedagem
- GitHub Pages: grátis, sem cobrança por uso, e o vídeo não passa por lá. Quem tiver o link assiste enquanto você transmite. O ID é aleatório e longo, então ninguém o adivinha.
- Descartados: Vercel (pedido seu) e artefato do Claude, que bloqueia captura de tela, WebRTC e conexão com servidores de fora.

## Riscos
- iPhone: só um teste real confirma que a música continua tocando.
- Os servidores grátis do PeerJS podem cair ou ficar lentos.
