# Trabalho 3 - DHCP e NAT 

![alt text](/img/image-34.png)


**Nota:**

+ Antes de começar, verifique em cada um dos terminais que o ficheiro ```/etc/dhcp/dhclient.conf``` existe e contém a linha  **send dhcp-client-identifier = hardware;**.  Caso contrário, crie-o (numa shell de root) e acrescente essa linha.

## Configurações inicias

### Terminais 

+ Devem ser configurados por DHCP
        
        1. ifconfig (verificar endereços)
        2. dhclient ```interface``` -v

    + Devem ter a correr os servidores de SSH (sshd) e de FTP (proftpd)
        
            (em ambos os terminais)
            
            Por a correr servidores SSH:
                systemctl status sshd
                (caso esteja desligado)
                systemctl start sshd
                systemctl enable sshd 

            Por a correr servidores FTP:
                systemctl status proftpd
                systemctl enable proftpd
                systemctl start proftpd



### Router Cisco (RCis)

+ A interface externa (f1/0) é configurada por DHCP. Pode ver que endereço lhe foi atribuído usando o comando  sh ip interface brief .


+ A interface interna (f1/1) é configurada estaticamente.

        1. enable 
        2. conf t 
        3. int f1/1 
        4. ip addr 172.16.0.1
        5. no shutdown
        6. copy running-config startup config

    + ```DHCP```

        + Pool de endereços a atribuir dinamicamente: 172.16.0.50 a 172.16.0.150


        + Endereço 172.16.0.11 reservado para o terminal 1

        + Além do endereço IP, deverão ser configurados por DHCP
            + Default gateway
            + Servidor de DNS (usar o 192.168.123.1)
            + Máscara de rede
            + Nome do domínio (usar o alunos.dcc.fc.up.pt)
        
        + Configure a duração das leases, que deverá ser sempre de 1 hora.

        ```Configuração RCis```  
        
                1. enable 
                2. conf t 
                3. ip dhcp excluded-address 172.16.0.1 172.16.0.11

> O professor a seguinte ```exlusão de endereços DHCP```

![alt text](/img/image-35.png)

                (dynamic é apenas o nome dado a esta pool)
                4. ip dhcp pool dynamic 
                    4.1 network 172.16.0.0 255.255.255.0
                    4.2 default-router 172.16.0.1
                    4.3 dns-server 192.168.123.1
                    4.4 domain-name  alunos.dcc.fc.up.pt
                    (bastava por lease 0 1, dias horas minutos)
                    4.5 lease 0 1 0
                    4.6 exit
                
                (manual é apenas o nome dado a esta pool)
> **Nota IMPORTANTE**:

> Como o ```Endereço Ip``` **172.16.0.11** calha ```dentro``` da ```pool dynamic```, os ```parâmtros da pool dynamic``` são ```herdados``` na ```pool manual``` , **exceto** a duração das ```leases```

                5. ip dhcp pool manual
                    5.1 host 172.16.0.11 255.255.255.0

>(verificar o ```MAC_TERM_1``` com ```ifconfig``` no ```Term1``` e ```remover os 00```: no inicio)

>(```0100```:(....) indica que é o ```client identifier MAC ADDRESS```)

                    5.2 client-identifier 0100:MAC_TERM_1 
                    5.3 client-name Term1
                    (bastava por lease 0 1, dias horas minutos)
                    5.4 lease 0 1 0

                6. Guardar configurações
                    6.1copy running-config startup-config

                7. Verificar pool
                    7.1 show ip dhcp pool

![alt text](/img/image-1.png)

                    7.2 show ip dhcp binding

![alt text](/img/image-2.png)
****

```Configuração Term1```
            
    (remover configuração dhcp de uma dada interface)
    1. sudo dhclient -r interface

    2. dhclient interface -v ou nmcli con up <interface>

    3. ifconfig 

![alt text](/img/image-3.png)



TODO: PEDIR AJUDA AO PROFESSOR NAS CONFIGURAÇÕES NAT 

+ ```NAT```
    + Fazer NAT usando o endereço da interface externa (f1/0) do router e activar overloading (PAT)

>Nota: utilizamos esta configuração que esta nos slides como base (não fizemos o que esta vermelho)
![alt text](/img/image-5.png)

```Configurações RCis```

    1. enable
    2. conf t
    3. int f1/0 
        3.1 ip nat outside
        3.2 exit
    4. int f1/1 
        4.1 ip address 172.16.0.1 255.255.255.0
        4.2 ip nat inside 
        4.3 exit

>**ip nat inside source list 10 interface f1/0 overload**

> ```ip nat inside source list 10``` faz NAT a todas as maquina na ```lista de acesso 10``` 

>```Endereço Ip``` utilizado para ```NAT``` vai ser aquele que é atribuido a ```interface f1/0``` (**ip nat outside** , ver conf na primeira /img/imagem deste ficheiro)

>```overload``` abaixo indica que é ```NAT com tradução de portas``` (aplica-se a todas as maquina na lista de acesso 10)

    5. ip nat inside source list 10 interface f1/0 overload


> **access-list 10 permit 172.16.0.0 0.0.0.255**

>Indica ```todas as maquinas``` da rede ```172.16.0.0/24``` e guarda na ```lista de acesso 10```

    6. access-list 10 permit 172.16.0.0 0.0.0.255
            
    7. copy running-config startup-config

**Notas**:
        
```ip nat outside source```:

    Traduz a origem dos pacotes IP que viajam de fora para dentro
            
    Traduz o destino dos pacotes IP que viajam de dentro para fora


+ Redireccionar (port forwarding) a porta 8022 da interface externa (f1/0) para a porta 22 (ssh) do terminal 1
        
    ```Port Forwarding em Cisco IOS```
            
        1. enable
        2. conf t 
        3. ip nat inside source static tcp 172.16.0.11 22 interface f1/1 8022
        4. copy running-config startup-config

        (Verificar)
        5. show ip nat translations
![alt text](/img/image-4.png)



>```Testar tradução configurações NAT```

1.  No ```RCis```

    * debug ip nat detailed

2. fazer terminal para um endereço ```FORA``` da rede local

    * Term1 $ ping -c 1 192.168.123.1


>Resultado (VERIFICAR AS TRADUÇÕES QUE OCORREM)
![alt text](/img/image-6.png)



 


### Router Linux (a configurar posteriormente)

duvida: quando é que é suposto configurar este Router??

+ ```DHCP```
    + Configuração idêntica à que tinha RCis
    + Duração máxima das leases de 2 horas.
    + Sempre que alterar o ficheiro /etc/dhcp/dhcpd.conf, teste esse ficheiro usando  dhcpd -t  antes de (re)iniciar o serviço.



> ```Configuração de endereços``` e ```Encaminhamento de pacotes```

    1. configuração da interface interna (ens4)
        1.1 nmtui
    	    a. Ipc4 config Manual
            b. Clicar em Addresses e colocar 172.16.0.1
            c. Remover o X em 'Never use this network for default route'
            d. Ativar automaticaly connect
            e. guardar e ativar interface ens4
        1.2 Confirmar 
            a. ifconfig

    2. Verificar se tem linha net.ipv4.ip_forward = 1 no ficheiro /etc/sysctl.conf



![alt text](/img/image-22.png)

> ```Configuração DHCP```

    1. $ nano /etc/dhcp/dhcpd.conf

        subnet  172.16.0.0 netmask 255.255.255.0 {
            range 172.16.0.12 172.16.0.254;

            option subnet-mask 255.255.255.0;
            option routers 172.16.0.1;
            option domain-name-servers 192.168.123.1;
            option domain-name "alunos.dcc.fc.up.pt";
            default-lease-time 1800;
            max-lease-time 7200;
        }

        host Term1 {
            hardware ethernet 00:12:f3:2d:58:00;
            fixed-address 172.16.0.11;
            option host-name "Term1";
            default-lease-time 604800;
            max-lease-time 604800;

        }

    2.  dhcpd -t 

    3. systemctl start dhcpd

    4. Ativar IP forwarding (colocar a linha abaixo no ficheiro /etc/sysctl.conf)

![alt text](/img/image-42.png)

    5. Verificação no terminal 1 
            
        (remover configuração dhcp de uma dada interface)
        5.1 sudo dhclient -r interface

        5.2 dhclient interface -v 

        5.3 ifconfig 

![alt text](/img/image-24.png)

+ ```NAT```
    + Configure as nftables para masquerading, isto é, source NAT usando o endereço (dinâmico) da interface externa. As regras criadas com o comando nftables são temporárias. Para as tornar persistentes, pode usar os seguintes comandos:

            nft list table nat > /etc/nftables/myNATtable.nft 
            echo 'include "/etc/nftables/myNATtable.nft"' >> /etc/sysconfig/nftables.conf 
            systemctl enable --now nftables 
            
    + Deve ter a correr o servidor de SSH.


![alt text](/img/image-25.png)

>```Configurações NAT```

+ Interface interna = **ens4**

+ Interface externa = **ens3**

+ nft add table ip nat

+ Configurar ```cadeia de prerouting``` para ```port forwarding```

        nft add chain ip nat prerouting { type nat hook prerouting priority -100\; }

+ Configurar ```cadeia de postrouting``` para ```mascaramento```
    
        nft add chain ip nat postrouting { type nat hook postrouting priority 100\; }

+ ``` Regras de NAT ```

    1. NAT para conexões de saída (masquerade)(```o que faz comando abaixo:``` um pacote que sai pela ```interface ens3``` e tenha ```endereço Ip origem``` que seja da gama ```172.16.0.0/24``` vai fazer o ```mascarade``` )
        
        
            nft add rule ip nat postrouting oif "ens3" ip saddr 172.16.0.0/24 masquerade

    2. Port Forwarding: Redirecionar porta 8022 para 172.16.0.11:22 (```o que faz comando abaixo:``` se um pacote entrar pela ```interface ens3``` e for destinado a ```porta 8022``` então vai fazer ```Portforwarding``` para ```172.16.0.11:22```)

            nft add rule ip nat prerouting iif "ens3" tcp dport 8022 dnat to 172.16.0.11:22


        

> ```Persistência das regras nftables```

        nft list table nat > /etc/nftables/myNATtable.nft
        echo 'include "/etc/nftables/myNATtable.nft"' >> /etc/sysconfig/nftables.conf
        systemctl enable --now nftables 


> ```Verificações``` em ```Rlin```

+ **nft list table nat**

![alt text](/img/image-26.png)

+ **nft -a list table nat**

![alt text](/img/image-27.png)


+ **nft list ruleset** (retirei da config do professor, confirmar na nossa se e igual)

![alt text](/img/image-43.png)

> ```Testes das configurações```

    Fazer um ping de Term1 para endereço na rede externa (neste caso fiz para o Term2 que tinha endereço 192.168.123.250)

![alt text](/img/image-28.png)