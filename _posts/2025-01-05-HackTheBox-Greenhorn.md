---
title: "Hack The Box - GreenHorn"
categories: [CTF, Hack The Box - Play]
tags: [EASY, Linux, Web, Gitea]
mermaid: true
image: ../assets/img/hackthebox/greenhorn/greenhorn.PNG
---

GreenHorn foi uma máquina excelente para treinar as minhas habilidades durante o pentest. Primeiro foi identificado o Pluck CMS. Ao investigar mais a fundo, descobri que na porta 3000 estava disponível o código-fonte do site, contendo credenciais de acesso. Em seguida, explorei uma vulnerabilidade na versão da aplicação, que permitiu um LFI (Local File Inclusion). Com isso, fiz o upload de um arquivo malicioso, o que proporcionou o primeiro acesso ao alvo.

Para escalar privilégios, encontrei um arquivo PDF com credenciais de acesso. Esse arquivo precisou ser "despixelado" para que o conteúdo pudesse ser lido corretamente. Após esse processo, obtive a senha do root, o que possibilitou a escalada de privilégios.

# Diagram

```mermaid
graph TD
    A[Host Enumeration] -->|Nmap Scan| B[HTTP Enumeration]
    B -->|Identify Pluck| C[Explore Pluck Profiler]
    C -->|Find Sensitive Files| D[Exploit Pluck]
    D -->|Obtain Credentials| E[Use Credentials for login pluck profile]
    E -->|RCE| F[Reverse Shell]
    F -->|Privilege Escalation| G[PDF credentials]
    G -->|Login Root| H[Root Credentials]
```


## Information Gathering

### Port Scan
---

- `nmap -Pn 10.10.11.25`
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled.png)


## Enumeration

### HTTP 80

---

- http://10.10.11.25/
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled202.png)
    
- Há um direcionamento para: greenhorn.htb
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled203.png)

- Edição do `/etc/hosts`
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled204.png)
    
- Acessando a página Web

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled205.png)
    
- Nome da possível aplicação:
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled206.png)
    

- Ao clicar em "admin" conseguimos entrar no campo de login
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled207.png)


### HTTP 3000    


- Na porta 3000 encontramos uma aplicação Gitea, contendo o versionamento da aplicação (código fonte).

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled208.png)
    

- Selecionando o botão "Explore" conseguimos encontrar o código fonte da página.
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled209.png)
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled210.png)
    
- Em `GreenHorn/data/settings` Podemos encontrar informações da aplicação.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled211.png)

    $sitetitle = 'GreenHorn';
    $email = 'admin@greenhorn.htb';

- Foi obtido também um hash:

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled212.png)

        <?php
        $ww = 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163';
        ?>


- Utilizando o [hashes.com](https://hashes.com) foi possível obter o valor do hash e encontrar a senha para o acesso.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled213.png)

## Exploitation

- Utilizando a porta 3000, foi utilizado a credencial encontrada para acessar a conta de admin. 

Foi encontrado uma vuln na versão do sistema pluck 4.7.18 na qual permite o FLI no parâmetro que não realiza a higienização do arquivo que está sendo incluído. 
caminho do parâmetro:`http://greenhorn.htb/admin.php?action=installmodule`

fonte: [fonte: https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC/blob/main/poc.py](https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC/blob/main/poc.py)
    
![Untitled](../assets/img/hackthebox/greenhorn/Untitled215.png)

Foi encontrado esse exploit, porém não foi utilizado.

No parâmetro informado (`http://greenhorn.htb/admin.php?action=installmodule`), pudemos encontrar um local para subir arquivos.

![Untitled](../assets/img/hackthebox/greenhorn/Untitled216.png)

- Foi selecionado um reverseshell.php para realizar o upload na aplicação e tentar o shell reverso.

![Untitled](../assets/img/hackthebox/greenhorn/Untitled217.png)

![Untitled](../assets/img/hackthebox/greenhorn/Untitled218.png)


- Após selecionar a aplicação, foi colocado o shell de ataque para ouvir na porta determinada no reverse shell.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled219.png)

- Na hora de fazer o upload do arquivo, foi encontrado um erro na execução.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled220.png)

Foi constatado que a aplicação só aceita o arquivo .zip.

- Convertendo o shell para .zip:

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled221.png)

- O upload foi realizado com sucesso.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled222.png)

Assim que foi realizado o upload do arquivo, foi obtida a shell inicial do alvo.
Na referencia a qual foi utilizada  para realizar o ataque, pode-se observar um caminho onde possivelmente estaria a shell.

![Untitled](../assets/img/hackthebox/greenhorn/Untitled223.png)

![Untitled](../assets/img/hackthebox/greenhorn/Untitled224.png)



- Não foi possível acessar o conteúdo do host com a conta www-data.
Sendo assim, vamos tentar utilizar a credencial encontrada ( `iloveyou1` ) para acessar o usuário junior.

![Untitled](../assets/img/hackthebox/greenhorn/Untitled225.png)

- Foi obtido sucesso ao acessar a conta.
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled226.png)

    d4f9ddc90d984468af21edc53b4fd0e5

## Priv Escalation

- Podemos verificar que há um pdf na conta.
    
    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled227.png)
    

- Após realizar o download do arquivo, foi possível observar que o arquivo está pixelado no local da senha.

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled228.png)

Para despixelar foi utilizado o [https://github.com/spipm/Depix.git](depix.py)

No entando, para utilizar o sistema, precisamos de transforma-lo em png. Para isso foi utilizado o site [https://tools.pdf24.org/en/extract-images?source=post_page-----e87e1cc07864--------------------------------](PDF24 Tools Extract PDF images)

    
![Untitled](../assets/img/hackthebox/greenhorn/Untitled229.png)

Resultado:
![Untitled](../assets/img/hackthebox/greenhorn/Untitled230.png)
    
    sidefromsidetheothersidesidefromsidetheotherside

- Utilizando a credencial encontrada:

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled231.png)

- Encontrando a flag:

    ![Untitled](../assets/img/hackthebox/greenhorn/Untitled232.png)

    064255efb0e058760e060b915e547f42

