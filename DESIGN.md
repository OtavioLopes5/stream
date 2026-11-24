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
- **Link fixo (atualizado em 2026-11-24):** você escolhe o nome do link (ex.: `?v=minha-live-4821`), e ele fica igual em qualquer navegador ou PC onde você digitar o mesmo nome. Sem nome escolhido, vale um código aleatório salvo no navegador. Por dentro, o ID no PeerJS ganha o prefixo `tela-ao-vivo-`, para não bater com outros apps que usam o mesmo servidor grátis.
- **Senha (mínimo 12 caracteres):** você define a senha da live. Sem senha válida, o botão Transmitir fica desativado. O endereço do seu PC no PeerJS é calculado a partir do nome do link **e** da senha (PBKDF2, 300 mil rodadas). Quem não sabe a senha não consegue chegar no seu PC, ver o seu IP, saber se a sua página está aberta, nem registrar o seu endereço para pegar a senha de quem assiste. O seu PC ainda confere a senha de novo antes de mandar qualquer coisa. Trocar a senha recarrega a página, inclusive outras abas suas no mesmo navegador, e muda o endereço, então quem só sabia a antiga cai.
- **Quem assiste:** digita o próprio nome e a senha uma vez, e os dois ficam salvos no navegador dela. Senha errada parece igual a você estar fora do ar ("Aguardando transmissão…", com a dica de tocar em Sair). O botão **Sair** apaga a senha salva e mostra a tela de entrada de novo. A página só aceita vídeo vindo do endereço do seu PC.
- **Lista de quem assiste:** a sua página mostra os nomes de quem entrou com a senha certa. O nome é o que a pessoa digitou, sem verificação, e aparece sempre como texto puro (até 30 caracteres).
- **Privacidade:** a página pede para o Google e outros buscadores não indexarem (`noindex`) e não manda o endereço completo para outros sites (`no-referrer`). O código no GitHub é público, mas não tem nada secreto: nome e senha ficam só no navegador. Os commits usam o e-mail anônimo do GitHub.
- **IP:** a conexão é direta, então quem tem o link e a senha vê o IP da sua internet (cidade e operadora aproximadas). Para esconder até dessas pessoas, use uma VPN enquanto transmite.
- **Script do PeerJS:** vem do unpkg com trava de integridade (SRI). Se o arquivo mudar lá, o navegador se recusa a rodar.
- **Limite conhecido:** otaviolopes5.github.io é o mesmo site para todos os seus projetos do GitHub Pages, que dividem o armazenamento do navegador, onde a senha fica salva. Uma falha em outro desses projetos poderia ler a senha. A solução seria um endereço próprio só para este projeto.
- **Rede bloqueada:** não há repasse. Os servidores de repasse grátis do PeerJS (`eu-0` e `us-0.turn.peerjs.com`) estão fora do ar (testado em 2026-11-24). Se as duas redes forem muito fechadas, a imagem não chega.

## Câmera sobreposta (adicionado em 2026-11-24)
- **O que é:** um botão "Câmera" na sua página liga e desliga a webcam a qualquer momento. É só vídeo: o microfone nunca é pedido.
- **Como chega:** a câmera vai como um segundo vídeo, separado do da tela. A página de quem assiste mostra esse vídeo por cima da tela. Juntar os dois num vídeo só no seu PC foi descartado, porque o Chrome desacelera abas em segundo plano para uns 1 fps.
- **Posição:** você define onde a câmera começa. Ao transmitir, sua página mostra uma prévia da tela, e você arrasta a câmera nela. Quem assiste pode arrastar depois, e a escolha dela vale até ela recarregar a página.
- **Tamanho e coordenadas:** a câmera ocupa 25% da largura da tela, e a posição vai em proporção da imagem (0 a 1). Assim ela fica no mesmo lugar em qualquer tamanho de tela.
- **Picture-in-picture:** mostra só a tela, porque o navegador só põe um vídeo na janela flutuante. Na tela cheia, a câmera aparece.

## Celular
- As duas páginas se ajustam a telas de celular. Arrastar funciona com o dedo, e os botões ficam fora das bordas e do entalhe do aparelho.
- Transmitir a tela só funciona no computador (Chrome ou Edge), porque os navegadores de celular não capturam a tela. No celular, a página avisa isso.

## Hospedagem
- GitHub Pages: grátis, sem cobrança por uso, e o vídeo não passa por lá. Para assistir, é preciso o link e a senha.
- Descartados: Vercel (pedido seu) e artefato do Claude, que bloqueia captura de tela, WebRTC e conexão com servidores de fora.

## Riscos
- iPhone: só um teste real confirma que a música continua tocando.
- Os servidores grátis do PeerJS podem cair ou ficar lentos.
