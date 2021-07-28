# Pikaboo

-> Feito por: !orelha</>

Bom dia/tarde/noite senhores, Vou contar o passo a passo que eu fiz para conseguir rootar a machine Pikaboo na htb!

Então vamos lá!

Primeiramente peguei o ip e usei o "nmap" para fazer a enumeração de portas para análise.
->Nmap -sS -r -v ip | tee portas.txt

![nmap](https://user-images.githubusercontent.com/69881757/126920418-dd42e07b-921a-4f9c-a62d-8bdef68bd8fd.png)

Por default tem uma aplicação web rodando na porta 80, com um servidor nginx!
Ao fazer o fuzzing da aplicação web, foi descoberto um diretório /admin, porém sem permissão para entrar, pois pedia um login e senha.

![autenticacao](https://user-images.githubusercontent.com/69881757/126920742-cf091c48-fa70-4705-9bf8-856b4cdb7e67.png)

Porém, ao saber que o servidor era um nginx, fui fazer uma pesquisa sobre, e encontrei uma ótima documentação, que me levou ao resultado.
link da documentação-> https://blog.detectify.com/2020/11/10/common-nginx-misconfigurations/

![blog1](https://user-images.githubusercontent.com/69881757/126920825-6eb3263a-f32a-487b-9fae-946ec062d012.png)

Então com o conhecimento adquirido, foi colocado em prática e a aplicação nos trouxe algo interessante.

![server](https://user-images.githubusercontent.com/69881757/126920887-f2509d43-db4e-4adc-90c0-1f7986af3b8f.png)


E analisando o server-status, é possivel ver um diretório chamado->/admin../admin_staging/index.php?page=/var/log/vsftpd.log

Apenas pela análise da url, percebe-se que tem uma vulnerabilidade chamada LFI(LOCAL FILE INCLUSION), dando-se a resposta da log FTP

![logftp](https://user-images.githubusercontent.com/69881757/126921208-468e074a-9ef6-4162-9800-55f390cfea42.png)

Pensei, será que é possivel fazer um log poison? 
Então ao pesquisar tive minha resposta-> https://book.hacktricks.xyz/pentesting-web/file-inclusion#via-vsftpd-logs

Então fui correndo ao ftp, coloquei a payload, deixei uma porta ouvindo, e atualizei a pagina aonde estava a log do ftp.

![reverse-shell](https://user-images.githubusercontent.com/69881757/126921323-883b5af3-24ba-4670-8586-3af971585ae7.png)

Analisando os cronjobs da maquina, foi possivel perceber algo estranho, um script chamado csvupdate_cron e  csvupdate

![crontab](https://user-images.githubusercontent.com/69881757/126922051-201f9090-9978-42d6-8300-2e0ddf0fca0b.png)

---

![csvupdate](https://user-images.githubusercontent.com/69881757/126922105-db8f9ae8-3781-477e-8ce5-21f3e03c684e.png)

Então na analise desses 2 scripts em perl, teve o entendimento que cada script que seria colocado no ftp só seria executado se fosse haver uma
extensão .cvs, e executária todo código a partir de um pipe para frente |,pois eles recebem 2 paramêtros, perl open() injection
-> https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88890543

Como jogar um arquivo no ftp, sem login e senha?

Procurando na maquina, foi possível encontrar um arquivo chamado settings.py, aonde foi encontrado usuario e senha para LDAP, então ao fazer a conexão
foi possivel ver um user e uma password em base64

![connetc-ldap](https://user-images.githubusercontent.com/69881757/126922431-9181e04b-0f6e-41a9-b5cf-3c4306ff9e9b.png)

Daí foi só fazer o login no ftp e fazer o bypass no script.
Foi criado uma reverse e enviado para dentro do ftp, na espera de 1 minuto, foi possivel pegar o tão esperado root.

![root](https://user-images.githubusercontent.com/69881757/126922543-2e3a734e-51e0-403d-b63b-611a745cdc97.png)

E foi isso, uma ótima maquina para aprendizagem!

