---
layout: post
title:  "Talk is cheap. Show me the money!"
date:   2018-01-22 00:00:00 +0000
categories: bug-bounty
tags: bug-bounty hackerone ubiquiti
---

Esta primeira publicação tem como objetivo principal estimular e incentivar a participação de profissionais de segurança, hackers e entusiastas nos tão famigerados programas de recompensa por falhas; os chamados Bug Bounty Programs. Aqui não terá nada muito técnico, mas isso já está no TODO-List para publicações futuras.

### O que é Bug Bounty?

Pra quem não sabe o que é um `Bug Bounty` Program, uma forma simples de explicar é que são projetos que te recompensam por descobrir vulnerabilidades e reportar de maneira **ética** para o desenvolvedor/vendor do produto ou para quem usa o mesmo. Essas recompensas podem ser o conhecimento da descoberta, muitas vezes com nome e contato profissional divulgado no hall da fama; porém a recompensa também podem ser monetária; e é aí que tudo começa a ficar mais interessante.

Existem diversas empresas que possuem seu programa de recompensa para falhas como o [Facebook](https://pt-br.facebook.com/BugBounty/), [Google](https://www.google.com/about/appsecurity/reward-program/) ou [Microsoft](https://technet.microsoft.com/pt-br/library/dn425036.aspx). Além do mais, existem plataformas que concentram dezenas de centenas de empresas diferentes, atuando como um intermediador, com é o caso do [HackerOne](https://www.hackerone.com/), e outras empresas que aderem a programas de recompensa porém com o intuito de revender as falhas para outros órgãos (aka governos) ou adicionar a algum produto específico de segurança.

### Critério de Seleção

Eu vinha a um tempo já acompanhando relatórios de Bug Bounty que são publicados na plataforma da [HackerOne](https://www.hackerone.com/), e sempre que via alguém ganhando alguns dólares reportando _XSS Reflected_ eu sempre pensava o mesmo: "Pultz! Bem que poderia ser eu". Até que eu resolvi arregaçar as mangas e tentar a "sorte" em algum programa.

Para a escolha do programa, eu avaliei alguns pontos analisando os reports públicos:

- Um programa que possuísse regras e escopo claro
- A responsividade do vendor quando alguém reportava alguma falha
- O valor médio pago por cada tipo de falha
- Algo que não fosse extremamente obscuro de testar (queria algo acessível)
- E, é claro, algo que eu considerasse extremamente divertido

### Escolhendo o Vendor

Como eu estava na vibe de estudar segurança em embarcados (aka IoT), eu consegui achar um vendor que encaixou como uma luva nos meus requisitos, a [Ubiquiti Networks](https://hackerone.com/ubnt). Eles possuem regras claras, escopo bem definido, pagam bem e analisar um firmware e achar falhas iria ser massa. Apesar de outros vendors pagarem bem, um diferencial da UBNT foi que eles pagam assim que a vulnerabilidade é confirmada, enquanto que outros vendos só pagam quando a falha é corrigida, o que é compreensível, porém pode levar mais de três meses e isso iria me matar de ansiedade.

### Definindo o Alvo

O próximo passo foi identificar o alvo e, depois de alguma pesquisa e de conversar com alguns conhecidos, cheguei na conclusão de comprar um [EdgeRouter X no MercadoLivre](https://produto.mercadolivre.com.br/MLB-811540935-ubiquiti-edgerouter-x-er-x-5-portas-rj45-_JM?source=gps), um roteador gerenciável e com algumas funcionalidades bem interessantes. A compra (com frete) me fez desembolsar em torno de R$ 300,00. Mas eu sei que seria para um bem maior, e se achasse um simples XSS, já ganharia em torno de U$ 150,00, que seria suficiente para recuperar o valor investido (além de um pequeno lucro).

![EdgeRouter X no MercadoLivre]({{ "/assets/images/posts/2018/01/edgerouter-ml.png" | absolute_url }})

Fiz a compra na quinta-feira, dia 30 de maio, e o router foi entregue quase duas semanas depois, no dia 14 de junho, véspera do feriado de 15 de junho (Corpus Christi).


### Testes e Resultados

Vou deixar claro que a "pornografia" técnica ficará em uma outra publicação futura. Além de que o objetivo dessa publicação é só relatar minha experiência e servir de incentivo aos que desejam ingressar na mesma área.

Bom. Continuando...

No dia que o roteador chegou, eu trabalhei até um pouco tarde, por volta de 21h e, então, resolvi sentar na minha máquina de pesquisas/estudos e dar uma olhada rápida na interface Web do roteador, já procurando pela possibilidade de identificar o meu tão precioso XSS.

Acontece que, analisando as mensagens HTTP, foi possível identificar <strike>na cagada</strike> uma vulnerabilidade em que um usuário de perfil Read-Only (eg: operator) conseguiria executar comandos como _root_ no dispositivo. Isso foi um achado maior do que o esperado, principalmente por ter sido somente nos primeiros minutos analisando a aplicação web do roteador. Porém, sem muita esperança, acreditei que a falha não teria tanto valor, já que seria necessário uma conta somente-leitura para o êxito da exploração, o que me fez considerar essa vulnerabilidade como uma simples Escalação de Privilégio, ao invés de uma Execução Remota de Comandos (RCE).

### Escrevendo o Relatório

Uma dica fundamental é: escreva o relatório da melhor forma e o mais completo que você conseguir, pois isso irá influenciar significativamente no tempo de resposta e pode principalmente influenciar na recompensa do seu achado.

Um relatório tem que ter por objetivo principal descrever a falha, deixar claro o impacto dela e, principalmente, ajudar/facilitar a reprodução da mesma. Quando os analistas/consultores da empresa lerem o relatório, quanto mais rápido eles entenderem a falha e conseguirem reproduzir ela, mais rápido ela passará pela triagem e mais rápido você vai receber a tão esperada recompensa.

É importante deixar bem claro tudo o que foi feito para reproduzir a vulnerabilidade. Se for o caso, restaure a aplicação (fresh install) e anote passo-a-passo do que foi preciso fazer até conseguir reproduzir a falha. No caso dos embarcados, é importante deixar claro a versão do hardware e a versão do firmware testada.

Um exemplo foi uma falha que reportei, porém a mesma só seria possível na primeira vez que se autenticasse na interface web. Não percebi esse detalhe e tive que trocar algumas mensagens com o vendor até conseguir chegar a um denominador comum e identificar esse critério na reprodução da falha.

Continuando ...

Na noite de quinta ainda, escrevi um relatório o mais detalhado possível, explicando cada um dos passos necessários para reproduzir a falha, e submeti no HackerOne (depois de revisar algumas dezenas de vezes).

Fui deitar e confesso que tentei dormir rápido, mas a ansiedade não me deixou, me fazendo ir até de madrugada pensando no relatório e em outras possíveis falhas ou testes que poderia fazer.


### A Primeira Recompensa

Ao acordar de manhã <strike>não tão cedo</strike>, fui checar meus e-mails no celular como de costume, e notei que já tinham dois e-mails na caixa de entrada referente ao HackerOne. De cara eu já pensei o pior: "Será que falta alguma coisa no processo?", "Será que esqueci de reportar algo?", "Será que é duplicado?" etc.

O primeiro e-mail foi que o meu report passou pela triagem, o que significa que a vulnerabilidade que eu reportei foi confirmada.

Já me dei por satisfeito, ficando contente com os possíveis U$ 250,00 que estariam por vir. Até que eu vi o seguinte e-mail, que simplesmente dizia que eu já havia sido pago. Isso mesmo! Em menos de 12 horas a vulnerabilidade que eu identifiquei foi confirmada e a recompensa foi paga. Além do mais, ao invés dos U$ 250,00 que eu esperava, eu fui **recompensado em U$ 1.500,00**, que além de pagar o investimento no EdgeRouter que eu havia comprado, daria para comprar mais 15 modelos iguais.

![Primeiro relatório de vulnerabilidade]({{ "/assets/images/posts/2018/01/ubnt-primeiro-report.png" | absolute_url }})

### Money, money e um convite inusitado(?)

Ganhar U$ 1.500,00 em uma brincadeira que levou poucos minutos foi mais que o suficiente para eu viciar na coisa. Lembram-se que era véspera de feriado? Pois então. Fiquei o feriado e o final de semana praticamente inteiro fuçando o aparelho, e o resultado foi mais quatro vulnerabilidades reportadas. Porém, diferente do primeiro report, uma delas foi considerada duplicada e os demais três foram recompensados em dois de U$ 150,00 e um de U$ 500,00. Isso se deve provavelmente à necessidade de iteração com o usuário e pelo fato da vulnerabilidade ter sido identificada em uma _feature_ beta do firmware.

Como eu peguei gosto pela coisa, mas não tinha muito tempo disponível fora dos finais de semana, decidi me acalmar um pouco e só olhar um pouco durante os finais de semana. Porém no dia 15 de junho eu recebi uma mensagem privada da Ubiquiti me convidando para testar um equipamento do Beta Store, o [EtherMagic](https://store.ubnt.com/products/ethermagic), e caso eu aceitasse, eles me enviariam um kit **gratuito**. Isso fez com que eu percebesse o reconhecimento da empresa frente às minhas pesquisas de vulnerabilidade dos equipamentos deles.

![Convite recebido]({{ "/assets/images/posts/2018/01/invite-beta-test.png" | absolute_url }})

### Não deixa o samba morrer...

Enquanto o aparelho novo não chegava, eu acabei olhando mais um pouco no bom e velho EdgeRouter durante mais um final de semana e o resultado foi a descoberta de mais duas vulnerabilidades. Como eu fiquei um pouco frustado com a recompensa anterior, resolvi voltar para o último release não-beta do firmware e o resultado foi interessante. Das duas falhas que havia reportado, uma delas me rendeu U$ 1.000,00 e a outra me rendeu mais U$ 1.500,00.

Vamos fazer as contas: U$ 1.500,00 do primeiro relatório, U$ 150,00 + U$ 150,00 + U$ 500,00 da segunda "remessa" de reports (queimei um report como duplicado), e depois mais U$ 2.500,00 em dois reports reportados no final de semana seguinte. Eu já havia, somente olhando no meu tempo livre, acumulado U$ 4.800,00. Como diz um amigo meu: "Já da pra pagar umas contas!". 

### Finalmente chegou!! Oops...

Depois de duas semanas que o aparelho novo foi postado, ele finalmente chegou nas minhas mãos. No começo fiquei um pouco preocupado de já abrir ele e sair soldando pinagem para UART (falarei sobre isso em publicações futuras) e acabar estragando, afinal de contas, se estragasse eu não poderia simplesmente abrir uma loja virtual e comprar outro. Mas infelizmente como o aparelho é beta, o suporte dele é bem limitado (pode-se dizer até que é inexistente), inclusive não existe uma versão do firmware dele na página de download do vendor. Ou seja, eu deveria extrair o firmware diretamente do device (também falarei disso em uma publicação futura) para conseguir acesso nele e fazer análise das aplicações em execução.

![EtherMagic]({{ "/assets/images/posts/2018/01/ethermagic.png" | absolute_url }})

Já era tarde da noite e eu estava razoavelmente cansado de tentar acesso ao aparelho. Porém eu, com meu conhecimento em hardware tendendo a zero, comecei a brincar com os comandos do bootloader dele (o U-Boot) e, de uma forma quase que milagrosa, consegui cagar com a memória FLASH do aparelho. #gênio 

Quando ele parou de responder e depois de um reboot o led sequer ascendia eu já previ o pior. Só me vinha na memória uma piada que eu sempre falava quando o assunto era mexer com hardware: "eu não mexo muito com hardware, porque não gosto de mexer com coisas que não da pra fazer backup". Enquanto meu coração quase parava de bater, meu socorro foi um analista de segurança da Ubiquiti que aceitou trocar o hardware dele com o meu para eu continuar com os testes e ele tentar de alguma forma resolver o problema do meu diretamente com os engenheiros.

### Talk is cheap. Show me the money!

Ao invés de tentar fazer uma análise botton-up (do firmware para a aplicação), eu tentei mudar o modus operandi para uma análise top-down. Ou seja, resolvi seguir pelo caminho mais simples e comecei a usar e analisar o aparelho sem ter o firmware em questão, somente com o que me era acessível. Ele implementava um protocolo específico que sequer era sob HTTP, portanto para a análise rolou um pouco de escovação de bits e muito wireshark/tcpdump.

Infelizmente como esse aparelho faz parte do programa Beta, a Ubiquiti não me autorizou fornecer mais detalhes sobre as falhas encontradas. Mas em um só final de semana reportei três falhas críticas no aparelho, as quais me renderam U$ 5.000,00 CADA UMA.

Após o pagamento dessa última remessa de recompensa infelizmente o programa de recompensa para o EtherMagic foi congelado e ele passou a ficar fora do escopo dos testes.

### Balanço Geral

Um resumo desse mês brincando com Bug Bounties foram 10 falhas reportadas, sendo uma delas duplicada e 9 falhas com rendimento total de U$ 19.800.00, o que é grana pra c@%$#$:

![Extrato do HackerOne]({{ "/assets/images/posts/2018/01/h1-extrato.png" | absolute_url }})

Também como resultado disso, eu fechei 2017 na quinta posição do ranking de agradecimento da Ubiquiti Networks:

![Ranking da Ubiquiti em 2017]({{ "/assets/images/posts/2018/01/ubnt-thanks.png" | absolute_url }})

### Conclusão

Apesar de não ter sido um artigo puramente técnico (isso é o que não vai faltar nesse espaço), espero que tenha valido a pena a leitura e que tenha estimulado mais a comunidade de InfoSec a se aventurar nesses programas de recompensa. Qualquer dúvida ou sugestão deixe seu comentário e, se possível curtam a publicação para que eu possa ter uma visibilidade maior de que tipo de conteúdo mais interessa meus leitores.

Boa caçada!

Maycon Vitali<br />
Hack N' Roll 