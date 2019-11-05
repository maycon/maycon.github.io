---
layout: post
title:  "Hacking Tricks: Identificando Portas de Saída para Shell Reversa"
date:   2019-11-05
categories: hacking-tricks
tags: hacking-tricks network pentest check-outgoing-ports
lang: br
ref: check-outgoing-ports
has-asciinema: false
---

Algumas vezes durante os projetos do dia-a-dia nos deparamos com acesso (ex: RCE) em que o host afetado que (aparentemente) não possui qualquer liberação de acesso à internet para a nossa boa e velha shell reversa. Dessa forma temos que procurar por alternativas para exfiltrar dados (ex: DNS) ou, antes disso, validar se realmente não temos qualquer acesso à internet.

Durante um teste me deparei com essa situação e, para aproveitar minha saudade de codar alguma tool, resolvi implementar uma ferramenta que aceitasse qualquer conexão que viessem (em qualquer porta). Dessa forma é possível identificar se o host comprometido possui alguma exceção no firewall da rede que permita ele sair para a internet.

No mais, boa leitura.


### Trick


Essa dica se resume basicamente a um código Python que responde `SYN+ACK` para qualquer pacote `SYN` que recebido. Utilizando a biblioteca Scapy do Python foi possível desenvolver essa técnica com muito pouco esforço, como pode ser visto abaixo:

``` python
#!/usr/bin/env python3

#
# SYN+ACK ALL - Written by Maycon Vitali
#

from scapy.all import *
import time

def PacketCallback(pkt):
    print (
        "[{}] SYN packet received from {} on port {}. Replying with SYN+ACK...".format(
                str(time.strftime("%Y-%m-%d %H:%M:%S")),
                pkt[IP].src,
                pkt[TCP].dport
        )
    )

    # Create the IP datagram exchanging the source and the destination
    ip  = IP(
        src=pkt[IP].dst,
        dst=pkt[IP].src
    )
    
    # Create the TCP datagrana exchanging the ports annd adjusting the flags
    # and the seq/ack numbers.
    tcp = TCP(
        sport=pkt[TCP].dport,
        dport=pkt[TCP].sport,
        flags='SA',
        seq=pkt[TCP].ack,
        ack=pkt[TCP].seq+1
    )

    # Do it!
    send(ip/tcp, verbose=False)

# Start the sniffing
sniff(
    iface="eth0",
    prn=PacketCallback,
    filter="(tcp[tcpflags] & tcp-syn != 0)"
)
```

Para que o código acima funcione apropriadamente, precisamos de dizer ao Kernel do Linux para não enviar pacotes `RST` quando m emso receber uma mensagem `SYN` em uma porta não aberta. Isso pode ser feito com o seguinte comando do `iptables`:

```
root@hacknroll:~# iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP
root@hacknroll:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  anywhere             anywhere             tcp flags:RST/RST
```

Tendo feito isso, basta executar o nosso script para começar a receber mensagens de conexão do além (aka Internet):

```
root@hacknroll:~# ./synack-all.py
[2019-11-05 14:05:12] SYN packet received from 117.xxx.xxx.160 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:41] SYN packet received from 193.xxx.xxx.46 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:41] SYN packet received from 193.xxx.xxx.46 on port 445. Replying with SYN+ACK...
[2019-11-05 14:05:48] SYN packet received from 45.xxx.xxx.87 on port 6052. Replying with SYN+ACK...
[2019-11-05 14:05:54] SYN packet received from 185.xxx.xxx.14 on port 4590. Replying with SYN+ACK...
[2019-11-05 14:06:05] SYN packet received from 14.xxx.xxx.191 on port 445. Replying with SYN+ACK...
...
```


Caso deseje capturar e responder somente para algum host/rede específico, basta adicionar os filtros correspondentes, como exemplificado abaixo:

``` python
sniff(
    iface="eth0",
    prn=PacketCallback,
    filter="(tcp[tcpflags] & tcp-syn != 0) and (src net 192.111.0.0 mask 255.255.0.0)"
)
```


Agora basta utilizar qualquer artifício que desejar para validar se o script está funcionando corretamente. Abaixo é possível que a porta 80 aparenta estar aberta:

```
$ ncat -z -v 68.xxx.xxx.181 80
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 68.xxx.xxx.181:80.
Ncat: 0 bytes sent, 0 bytes received in 0.24 seconds.
```

Com pouco esfoço é possível varrer todas as portas e validar quais portas podem sair para a internet e, então, utilizá-las para obter a sua tão sonhada shell reversa:

``` bash
$ for PORT in $(seq 65535); do ncat -z -v 68.xxx.xxx.181 $PORT 2>&1 | grep Connected; done
Ncat: Connected to 68.xxx.xxx.181:80.
Ncat: Connected to 68.xxx.xxx.181:443.
Ncat: Connected to 68.xxx.xxx.181:3389.
Ncat: Connected to 68.xxx.xxx.181:8080.
```

No caso acima, as portas 80, 443, 3389 (rdp) e 8080 possúem acesso à internet.

### Conclusão

That's all folks!


Hack N' Roll

### Referências
- [Scapy Project](https://scapy.net)