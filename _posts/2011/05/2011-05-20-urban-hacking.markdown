---
layout: post
title:  "Urban Hacking"
date:   2011-05-20
categories: hacking
tags: urban hacking-tricks gold
lang: br
ref: urban-hacking
has-asciinema: false
---

Estava conversando com o João Victor (@jvrmaia) e ele me disse que o pessoal do suporte (UFES) adiquiriu uma Câmera IP para fazer o monitoramento do prédio.

Quando eles foram procurar as documentação de instalação da câmera, tiveram uma surpresa ao ver que diversas câmeras são ligadas publicamente à internet e com senha padrão (ou pior ainda, sem senha).

Com a mente aguçada em saber se isso é bastante comum, fui na página da Toshiba, catei o manual de uma das Câmeras IP dela e no manual vi algumas figuras da interface Web de gerenciamento/visualização e, como imaginei, encontrei diversos IP’s públicos com acesso sem autenticação as câmeras.

Um exemplo comum foi com a seguinte pesquisa do Google:
[intitle:”network camera – User Login”](https://www.google.com.br/search?q=intitle%3A%22network+camera+-+User+Login%22)

Onde o primeiro resultado foi uma Câmera IP de (aparentemente) de um porto, onde é possível até movimentá-la (isso foi divertido):

![Primeiro relatório de vulnerabilidade]({{ "/assets/images/posts/2011/05/Captura_de_tela-TOSHIBA-Network-Camera-User-Viewer-for-Single-Screen-Display-Chromium-300x267.png" | absolute_url }})

Bom, agora eu me pergunto quando as pessoas vão passar a realmente se preocupar com esse tipo de coisa. Muito pode ser feito com esse tipo de informações, principalmente porque as câmeras são posicionadas estratégicamente em direção onde temos maior risco de incidente.

Podemos até saber se a locadora do Tio Phill está com fila muito grande antes de ir la: http://96.2.189.42:8080/ (user: `root` / pass: `ikwb`).