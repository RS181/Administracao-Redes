# Trabalho 3 - DHCP e NAT


> **Notas**

+ Para parar o ```debug ip nat detailed``` basta fazer no router em que foi ativado o comando ````no debug ip detailed``` (mesmo que estejam a **receber mensagens enquanto digitamos**)

## 1. Nos terminais 1 e 2 desactive a interface de rede ( nmcli con down ens3 ) e limpe a cache do dhclient com  rm -f /var/lib/{dhclient,NetworkManager}/*.lease . Limpe também em RCis eventuais leases anteriores com  clear ip dhcp binding * .

```Terminal 1 e 2```

    nmcli con down ens3

    # limpar cache dhcp
    rm -f /var/lib/{dhclient,NetworkManager}/*.lease


```RCis```

> **Nota**: só apaga binding dinâmicos (aqueles que foram configurados estaticamente ou manualmente ficam)
    clear ip dhcp binding * 

### a. Inicie uma captura wireshark na interface f1/1 de RCis e volte a activar a interface ens3 do terminal 1. Repita o procedimento para o terminal 2. As capturas devem mostrar o conteúdo das mensagens. (2 × capRes)


```Terminal 1```

    nmcli con up ens3

![alt text](/img/image-7.png)


+ ```DHCP Discover``` (Usada pelo cliente para descobrir servidor(es) DHCP existente(s), ```neste momento não tem endereço ip atribuido``` a unica coisa que a identifica é o ```MAC address```)

![alt text](/img/image-8.png)

+ ```DHCPOFFER``` (Usada para um servidor ```oferecer``` uma ```configuração``` (```Endereço IP```) a um cliente)

![alt text](/img/image-9.png)

+ ```DHCPREQUEST``` (Usada pelo cliente para ```pedir``` uma dada ```configuração``` (```Endereço IP```))

![alt text](/img/image-10.png)

+ ```DHCPACK``` (Usada pelo servidor para ```confirmar``` a ```atribuição da configuração``` )

![alt text](/img/image-11.png)

```Terminal 2```

    nmcli con up ens3

![alt text](/img/image-12.png)


+ ```DHCP Discover```  (Usada pelo cliente para descobrir servidor(es) DHCP existente(s), ```neste momento não tem endereço ip atribuido``` a unica coisa que a identifica é o ```MAC address```)

![alt text](/img/image-13.png)

+ ```DHCPOFFER``` (Usada para um servidor ```oferecer``` uma ```configuração``` (```Endereço IP```) a um cliente)

![alt text](/img/image-14.png)

+ ```DHCPREQUEST``` (Usada pelo cliente para ```pedir``` uma dada ```configuração``` (```Endereço IP```))

![alt text](/img/image-15.png)

+ ```DHCPACK``` (Usada pelo servidor para ```confirmar``` a ```atribuição da configuração``` )

![alt text](/img/image-16.png)

### b. Apenas a partir das mensagens capturadas e respectivo conteúdo é possível saber qual dos dois tem um endereço fixo


R: Não porque o DHCP apenas é um protocolo que atribui endereços, esse ```conceito de ser fixo ou não é interno a propria maquina```, e ```indeferente para o dhcp```



## 2. No router RCis active o debugging do NAT com o comando  debug ip nat detailed . No terminal 1, faça download do ficheiro http://193.137.35.140/favicon.ico (use o wget). Referindo-se explicitamente às linhas de output de RCis, explique o trabalho do NAT (outRes + texRes).


```RCis```

    debug ip nat detailed

```Terminal 1```

    wget http://193.137.35.140/favicon.ico

(nossa explicação)

![alt text](/img/image-18.png)

(explicação do professor)

> Pedido de ```estabelicimento de conexão``` de ```dentro``` para ```fora``` (Term1 para rede externa)

![alt text](/img/image-36.png)


> ```i``` indica que uma ```pacote``` entrou pela ```interface interior```

> indica que (172.16.0.11,35910) **-->** (193.137.35.140,80). Adiciona na ```tabela NAT``` uma entrada a indicar que ```172.16.0.11``` (**Inside Local**) vai ser subsituido por ```193.137.35.140``` (**Inside Global**) e o router tenta utilizar , se possivel a mesma ```porta``` que **Inside Global** (neste caso 35910 esta livre e pode utilizar, e não é necessário fazer ```tradução de porta```.Caso contrário teria de ser feito essa tradução também)

![alt text](/img/image-37.png)


> Feita ```tradução``` ```172.16.0.11``` --> ```192.168.123.230``` (pacote vai para o exterior apartir da rede interna). ```Inside local``` --> ```Inside global```

> ```Endereço destino``` mantém-se (não faz sentido mudar-lo)

> Tradução guardada na ```Tabela NAT```

![alt text](/img/image-38.png)


> ```o``` indica que uma ```pacote``` entrou pela ```interface exterior```

> (193.137.35.140,80) --> (192.168.123.230,35910)

> Vem do ```servidor web``` para o endereço ```NAT``` ```Inside Global```

![alt text](/img/image-39.png)



>  Feita ```tradução``` ```192.168.123.230``` --> ```172.16.0.11``` (pacote vem do exterior para a rede interna). ```Inside global``` --> ```Inside local```

> Consulta ```tabela NAT```

![alt text](/img/image-40.png)

> Restantes pacotes seguem o mesmo processo



## 3.Desactive a interface de rede do terminal 2, desligue-o do SW2 e ligue-o no SW1, e volte a activá-la. Veja que endereço lhe foi atribuído (deve ser da rede 192.168.123.0/24).Do terminal 1, faça um ssh para o terminal 2 ( ssh ar@<ip_term2> ). Nessa shell, corra o comando  while true; do echo hello; sleep 1; done  para que o servidor ssh vá gerando pacotes. Explique o que acontece quando usa o comando  clear ip nat translation *  e comente a afirmação Quando usamos NAT com tradução de portas, os fluxos de pacotes passam de connectionless para connection-oriented. (texRes)

>```NOTAS:```

>(1) A conexão deve cair. Eventualmente, pode ter que correr mais que uma vez o comando para limpar os mapeamentos NAT, pois o que acontece depende da ordem dos pacotes de entrada e saída.

>(2) Para perceber melhor o que se passa, pode fazer capturas nas duas interfaces de RCis.


    Desactive a interface de rede do terminal 2, desligue-o do SW2 e ligue-o no SW1, e volte a activá-la

```Terminal 2``` 
        
        nmcli con down ens3

![alt text](/img/image-19.png)

        nmcli con up ens3

****
    Veja que endereço lhe foi atribuído (deve ser da rede 192.168.123.0/24)

![alt text](/img/image-20.png)

    Do terminal 1, faça um ssh para o terminal 2 ( ssh ar@<ip_term2> ).

    Nessa shell, corra o comando  while true; do echo hello; sleep 1; done  para que o servidor ssh vá gerando pacotes. 

    Explique o que acontece quando usa o comando  clear ip nat translation *.
```RCis```

    1. enable
    2. clear ip nat translation * 

    R: Quando executamos no router RCis clear ip nat translation * , ele limpa todas as entradas da tabela NAT .

        Impacto:

        ---> A conexão SSH entre os terminais é interrompida imediatamente.

        ---> Isso ocorre porque os pacotes subsequentes não encontram uma tradução válida no NAT e, portanto, não podem ser roteados corretamente entre as redes privada e pública.

        ---> O fluxo gerado pelo comando while true; do echo hello; sleep 1; done também será interrompido, pois depende da conexão SSH.

    comente a afirmação Quando usamos NAT com tradução de portas, os fluxos de pacotes passam de connectionless para connection-oriented.

(nossa respota)
R:Esta afirmação é parcialmente verdadeira, mas precisa de esclarecimento:
    
+ ```NAT``` por si só **não transforma** ```protocolos connectionless``` em ```connection-oriented```:

    + NAT opera no nível de camada 3 (IP) e camada 4 (TCP/UDP).
    
    + **Protocolos como UDP continuam sendo connectionless, mesmo passando pelo NAT.**
        
+ A ```tabela NAT``` introduz um **aspecto de estado** que **simula conexão**:

    + Para **traduzir** os ```endereços``` e ```portas``` corretamente, o ```NAT``` mantém uma ```tabela de estado```, que **associa fluxos de pacotes**.

    + Por isso, do **ponto de vista** do ```NAT```, mesmo **fluxos UDP** (```connectionless```) **aparentam ser** "```connection-oriented```", pois ele **requer informações de início e fim do fluxo.**


(resposta do professor)

R: Quando usamos ```NAT com tradução de portas``` temos **obrigatoriamente** ```entradas por fluxo``` no ```router NAT```. Ao ter **estas entradas** temos algo muito ```semelhante``` ao ```encaminhamento orientado a conexões``` (```fluxos``` passam de ```connectionless``` a ```connection-oriented```).


## 4. Ainda com o terminal 2 ligado no SW1, faça um ssh desse terminal para o terminal 1


### a. Indique o comando utilizado

> **Nota**: 

+   para fazer ```SSH``` de ```fora``` para ```dentro```, temos de utilizar como ```endereço de destino``` o ```endereço``` ```Inside Global```, ou seja, o endereço do NAT (no caso endereço da interface externa)

+ utilizamos ``` -p 8022``` porque definimos a tradução ```(192.168.123.230,8022) -> (172.16.0.11,22)``` (ver indicação abaixo)

![alt text](/img/image-41.png)

        R: Term2 $ ssh ar@192.168.123.230 -p 8022


### b. Identifique uma situação em que é necessário fazer port forwarding usando uma porta não-standard. 

(nossa resposta)

```Cenário: Restrição no ISP (Provedor de Internet)```

+ Situação: Muitos provedores de internet **bloqueiam portas padrão** como a ```porta 80``` (HTTP) e a ```porta 443``` (HTTPS) em conexões residenciais para evitar que os usuários hospedem servidores web diretamente. Isso é feito por questões de segurança ou para forçar os usuários a contratar planos empresariais.

+ Solução com ```Port Forwarding```:

    + Suponha que você está hospedando um ```servidor web local``` na ```porta 80``` do seu **computador na rede interna**.

    + No ```roteador```, **configure** um redirecionamento de porta (```port forwarding```) da ```porta pública 8080``` para a ```porta interna 80``` no **IP local do servidor**.

    + Agora, **usuários externos** podem acessar o ```servidor we```b usando a ```porta não-standard 8080```, por exemplo: **http://<seu_ip>:8080**.


(resposta do professor)

R: Quaisquer situações em que a ```porta standard``` esteja ocupado por ```outro port forwarding``` 

+ E.g: se tivermos ```dois servidores SSH``` na ```rede interna```, naturalmente a ```porta 22``` de ```router NAT``` so podia estar a fazer o ```forwarding``` para **um desses servidores**.O outro teria que usar outra porta não standard.

+ E.g: quando o ```Router NAT``` implementa esse serviço, ou seja, se ```Router NAT``` tiver um ```servidor SSH``` a **correr no prorprio**, não podiamos usar essa porta.Como tal temos de usar uma porta não standard


## 5. Substitua na rede o RCis pelo RLin: desligue RCis de SW1 e SW2 e ligue a interface ens3 de RLin ao SW1 e a interface ens4 ao SW2 e faça no RLin uma configuração idêntica à configuração inicial do RCis. O terminal 2 deve continuar ligado ao SW1.

![alt text](/img/image-21.png)

>Configurações do ```RLin``` estão no ficheiro **config_inicias.md**


> ```Notas:``` 
+ Certifique-se de que o servidor de ftp está a correr nos terminais 1 e 2 e que não está a correr no RLin (use os comandos ```netstat -ant``` para ver se tem algum processo na porta 21 e ```systemctl start|stop|status proftpd``` para **activar**, **desactivar** ou **verificar** o estado do servidor FTP). 

+ Ponha o wireshark a capturar em todas as interfaces do RLin.

+ Em todas as alíneas desta pergunta deve correr o ftp em modo activo (use ftp -A ). Para melhor compreender o que se passa, atente aos endereços IP dos pacotes capturados e procure nesses pacotes os comandos PORT (ou EPRT) do FTP.



### a. Do terminal 2, tente fazer um ftp para o terminal 1 e fazer download (get) de um ficheiro. Justifique o insucesso deste procedimento. (capRes + texRes)

```FTP do Term1 para Term2```

![alt text](/img/image-29.png)

```FTP do Term2 para Term1 (TEMOS DE USAR IP PUBLICO DO ROUTER)```
![alt text](/img/image-44.png)


(resposta professor)

R: Porque no ```RLIN``` não temos ```nenhuma regra``` para a ```porta 21``` (ou seja, ```não definimos port forwarding para term1```).Quando chega um pedido ao ```router NAT``` (RLIN) ele acha que o pedido ftp é para ele, mas como não tem serviço a correr na ```porta 21``` , devolve **Connection refused**.


### b. No router Linux, redireccione (port forwarding) a porta 21 da interface exterior (ens3) para a porta 21 (ftp) do terminal 1. (confRes)

> Definimos uma ```regra de port forwarding``` no ```Router NAT``` (RLIN), que indica que todos os pacotes que vem do ```exterior``` para a ```porta 21``` e para o endereço ```Inside Global``` devem ser ```'reencaminhas'``` para  ```172.16.0.11 na porta 21```

    nft add rule ip nat prerouting iif ens3 tcp dport 21 dnat to 172.16.0.11:21

> Verificação no ```RLIN```

![alt text](/img/image-46.png)


### c. Tente novamente fazer o ftp e transferir um ficheiro. Comente os resultados, referindo o que acontece nas conexões de controlo e de dados. (capRes + texRes)


```FTP do Term2 para Term1 (TEMOS DE USAR IP PUBLICO DO ROUTER)```

![alt text](/img/image-48.png)

>Observação sobre captura **interface externa** de ```RLin```

+ ```Pedidos```

    + **Origem** ```192.168.123.250``` (**Term2**)

    + **Destino** ```192.168.123.40``` (**Endereço ip do router NAT**, ```Inside Global```)

+ ```Respostas```

    + **Origem** ```192.168.123.40``` (**Endereço ip do router NAT**, ```Inside Global```)

    + **Destino** ```192.168.123.250``` (**Term2**)



>Observação sobre captura **interface interna** de ```RLin```

+ ```Pedidos```

    + **Origem:** ```192.168.123.250``` (**Term2**)

    + **Destino** ```172.16.0.11``` (**Term1**, ```Inside Local```)

+ ```Respostas```

    + **Origem** ```172.16.0.11``` (**Term1**, ```Inside Local```)

    + **Destino**  ```192.168.123.250``` (**Term2**)

### d. Tente agora fazer um ftp do terminal 1 para o terminal 2 e fazer download de um ficheiro. Comente os resultados. (capRes + texRes)

```Term1 -> Term2```
![alt text](/img/image-49.png)



### e. No router Linux, faça download do ficheiro ftp-ct.nft (pode usar o wget) e active o suporte para ftp correndo os comandos:

> **Nota**: este ponto permite resolver o ponto anterior (para que o ```ftp``` possa funcionar de ```dentro``` para ```fora```).No **sentido contrário** ```não há problema```

>Não fiz esta parte (porque acho que não achei essencial fazer na altura).Se quiser fazer ver video

+ modprobe nf_nat_ftp
+ nft -f ftp-ct.nft

NOTA: Pode ver aqui como usar conntrack helpers com nftables.

Tente novamente o ftp da alínea anterior e comente os resultados tendo em conta o que acontece na conexão de controlo e na de dados. (capRes + texRes)