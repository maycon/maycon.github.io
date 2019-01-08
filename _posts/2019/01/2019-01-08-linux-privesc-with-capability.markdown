---
layout: post
title:  "Hacking Tricks: Escalação de Privilégio em Linux com Capability"
date:   2019-01-08
categories: hacking-tricks
tags: hacking-tricks privilege-escalation linux capability
lang: br
ref: linux-privesc-with-capability
has-asciinema: false
---

Dizem que quando você adentra em uma máquina seja durante um pentest, durante um estudo ou outra coisa (haha) você já pode ser considerado root/SYSTEM. Que mesmo estando com um usuários com pouco ou nenhum privilégio, existem diversas maneiras de se escalar para a autoridade máxima daquele host (sem entrar em detalhes pedantes). Essa escalação de privilégio pode ser feito através de vulnerabilidades no kernel, por falhas de aplicativos ou pelo simples desleixo/vacilo do sysadmin da rede. Com isso estou dando inicio a uma serie de _hacking tricks_ que são utilizados em pentests, competições de CTF etc. E a primeira publicação será sobre escalação de privilégios com _Linux Capabilities_. Boa leitura!

### O que são _capabilities_

Existem basicamente dois tipos de privilégios de processos em um sistema Linux, os processos privilegiados, com *user id 0 (root)*, e os processos não privilegiados, com *user id* diferente de zero. Processos privilegiados conseguem bypassar todas as checagens de permissões do kernel enquanto os processos não root precisa que todo e qualquer recursos que ele for acessar seja validado.

A partir do kernel 2.2 (sim, a muito tempo atrás), o Linux introduziu uma outra divisão de privilégios além das permissões do processo, essas podendo ser ativadas ou desativadas independente. Essa divisão de privilégios chama-se _capabilities_.


### Como funciona?

Para o resito de escalação de privilégio, podemos iniciar enumerando todas as _capabilities_ disponível no sistema. Isso é feito através do comando `getcap`, que pode ser executado para buscar de maneira recursiva (`-r`) a partir da raiz (`/`) do sistema de arquivos:

```
maycon@hacknroll:~$ /sbin/getcap -r / 2>/dev/null
/usr/bin/rsh = cap_net_bind_service+ep
/usr/bin/rlogin = cap_net_bind_service+ep
/usr/bin/dumpcap = cap_dac_override,cap_net_admin,cap_net_raw+eip
/usr/bin/cat = cap_dac_read_search+ei
/usr/bin/ping = cap_net_raw+ep
/usr/bin/rcp = cap_net_bind_service+ep
/usr/lib/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

Nota-se que antes o binário `ping`, que geralmente era `suid`, hoje ainda pertence ao _root_, porém sem o _suid bit_ habilitado: 
```
CAP_NET_RAW
      * Use RAW and PACKET sockets;
      * bind to any address for transparent proxying.
```

O que faz com que esse binário seja possível montar e enviar pacotes _raw_ na rede é a _capability_ `cap_net_raw`, como podemos ver na _man page_:
```
maycon@hacknroll:~$ ls -la /usr/bin/ping
-rwxr-xr-x 1 root 60112 Jul 12 01:52 /usr/bin/ping
```

Um outro exemplo é o `/usr/bin/dumpcap`, que é utilizado pelo `wireshark`. Essa aplicação que valida se o usuário pertence ao grupo `wireshark` para, então, permitir que o mesmo capture o tráfego de rede sem precisar do `sudo` ou de escalar para _root_.

Na listagem acima é possível notar que temos o `/usr/bin/cat` com a `capability cap_dac_read_search`. Para garantir o entendimento desse atributo, vamos ver o que a _man page_ diz sobre ele:
```
CAP_DAC_READ_SEARCH
      * Bypass file read permission checks and directory read and
        execute permission checks;
      * invoke open_by_handle_at(2);
      * use the linkat(2) AT_EMPTY_PATH flag to create a link to a
        file referred to by a file descriptor.
```

Como pode ser visto, temos uma falha de segurança, pois o `cat` possui a _capability_ de _bypassar_ a checagem de permissão de leitura feita pelo kernel. Dessa forma mesmo sem `suid bit` ou sem ser _root_ é possível ler qualquer arquivo utilizando o mesmo:

```
maycon@hacknroll:~$ cat /etc/shadow | grep root
root:$6$X.E.toNpWI9G4IL3$8gwwSdv...snip...Ky1duJ9xP6c94Lg.:17870::::::
```

É claro que o `cat` foi posto de maneira proposital para exemplificar. Porém as capabilities do `/usr/bin/dumpcap`, por exemplo, são padrões nos ambientes que possuem _wireshark_ instalado (eg: Kali Linux). Então fica como exercício para casa descobrir uma forma de escalar privilégio em ambientes cujo seu usuário é não privilegiado e está no grupo _wireshark_ (hint: `cap_dac_override` vai te ajudar).

### Conclusão

Essa foi uma escrita rápida para demonstrar uma forma bem interessante de escalar privilégio em sistemas Linux. Espero poder ter tempo para escrever outros _hacking tricks_ e contribuir de alguma forma com quem está iniciando. Quem quiser brincar mais com isso basta ler a _man page_  e utilizar os comandos `getcap` e `setcap`. 

Para dúvidas, sugestões ou reclamações basta utilizar o espaço de comentários abaixo ou os contatos disponíveis na barra lateral do site.

Hack N' Roll

### Referências
- [man 7 capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html)