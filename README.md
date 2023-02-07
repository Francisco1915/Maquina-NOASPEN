# Maquina-NOASPEN
Teste de Penetração à máquina fornecida pelo professor da unidade curricular Testes de Penetração e Hacking Ético

Avaliação: 19,5

## Enumeração

### Enumeração dos serviços:
```bash
nmap -A 192.168.57.3
```
![image](https://user-images.githubusercontent.com/83584144/210282553-e9155171-f8b1-4031-9efc-24c46f8edd1a.png)

Como podemos ver pelo scan do nmap, temos o ssh na porta 22, http na porta 80 e 8080 (Tomcat) e também o samba nas portas 139 e 445.

### Enumeração smb:
```bash
enum4linux 192.168.57.3
```
#### Serviços a correr neste protocolo:

![image](https://user-images.githubusercontent.com/83584144/210282678-9b8aaf74-aff2-4074-8c83-c3baca4e4fb4.png)

#### Users:

![image](https://user-images.githubusercontent.com/83584144/210282684-2f14fb1d-cc40-401e-97a2-726280b3322a.png)

#### Shares:

![image](https://user-images.githubusercontent.com/83584144/210282690-f3fd22f5-df5c-4630-93f8-b34648fb3215.png)

## Analise da máquina e Mapeamento de vulnerabilidades:

### Web servers:

#### Porta 80:

![image](https://user-images.githubusercontent.com/83584144/210282705-032cd737-dbe3-4a83-922a-5801aac3dfa8.png)

Temos um html simples que nos diz que o site se encontra em manutenção.

#### Porta 8080:

![image](https://user-images.githubusercontent.com/83584144/210282724-1d3ba238-c5e7-46f0-8e0c-bdba0b491f5e.png)

Página inicial do tomcat version 9.0.7.

### Nessus:

![image](https://user-images.githubusercontent.com/83584144/210282730-3c5f3594-4247-4f5b-88b1-49e0138ddb3f.png)

No nosso scan do nessus encontramos algumas vulnerabilidades.

![image](https://user-images.githubusercontent.com/83584144/210282733-8b74fdb9-d07a-48fb-9352-908a558d1639.png)

### Nuclei:

Analise de cada serviço http da máquina:

#### Porta 80:
```bash
nuclei -u http://192.168.57.3:80/
```

![image](https://user-images.githubusercontent.com/83584144/210282748-38834c6e-cda2-421c-b408-1fe81a684978.png)

Sem informação relevante…

#### Porta 8080:
```bash
nuclei -u http://192.168.57.3:8080/
```

![image](https://user-images.githubusercontent.com/83584144/210282752-d0c704d9-bd57-46c5-a7f8-c42eaaae8c57.png)

Podemos ver que encontrou uma vulnerabilidade critica, está também já encontrada no scan do nessus.

### Nikto:

Scan para tentar encontrar diretórios dos serviços http.

#### Porta 80:
```bash
nikto-h http://192.168.57.3:80/
```

![image](https://user-images.githubusercontent.com/83584144/210282762-a51d581b-c8ee-40a3-9c17-5bdf29b42b2a.png)

Encontramos o /development/, o que já era de esperar já que o html inicial da página diz que está em manutenção.

#### Porta 8080:
```bash
nikto-h http://192.168.57.3:8080/
```

![image](https://user-images.githubusercontent.com/83584144/210282771-36dac56d-4607-471d-bb18-e32b565adc9b.png)

Encontrou vários diretórios, mas ao tentar aceder aos mesmos, era pedido as credenciais de acesso o que tornou esta informação um bocado descartada.

## Exploração de vulnerabilidades

### CVE-2020-1938:

Vulnerabilidade encontrada no nessus e nuclei.

Esta vulnerabilidade é conhecida com GhostCat -Apache Tomcat AJP File Read/Inclusion Vulnerability.

Exploit da vulnerabilidade:

![image](https://user-images.githubusercontent.com/83584144/210282777-43bea444-bafe-4f83-b2b4-cb76953948fc.png)

Pode se concluir que esta vulnerabilidade não é muito vantajosa para o nosso objetivo…

### /development:

![image](https://user-images.githubusercontent.com/83584144/210282784-6b30dbe1-034b-44dc-8744-915ae4bbad1f.png)

#### Desenvolvimento.txt:

![image](https://user-images.githubusercontent.com/83584144/210282795-ebb5bb5d-b887-4787-b4fb-5cbe32a73c37.png)

Podemos ver a troca de msg entre dois utilizadores que em princípio serão o K de Kay e J de Jan, os users encontrados no ponto da enumeração.

#### J\_notas:

![image](https://user-images.githubusercontent.com/83584144/210282798-13d90c50-1cd2-45de-93ac-c736a294204b.png)

Mensagem enviada pelo Kay para o Jan a dizer que fez uma auditoria e aconselha o Jan a mudar de password pois a atual é fraca e não esta de acordo com a política de passwords, sendo assim ficamos a saber que o user Jan tem uma password fraca o que pode facilitar um brute force já que temos a porta 22 (ssh) aberta.

### SMB share /Anonymous:

![image](https://user-images.githubusercontent.com/83584144/210282803-05a29050-662e-48e9-bf86-bc628e6e68db.png)

![image](https://user-images.githubusercontent.com/83584144/210282805-3169df26-e039-49cb-9be2-b475bea65565.png)

Podemos ver que o Jan não é la um bom funcionário 😊.

Pelo que a mensagem transmite estive à procura de algum ficheiro que não devia estar lá, mas infelizmente não encontrei nada.

### Brute force user Jan:

Com a informação retirada pelos txts encontrados podemos partir do inico que a password do Jan é fraca sendo assim vamos realizar um brute force com a lista mais conhecida "rockyoulist.txt".
```bash
hydra -l jan -P /usr/share/wordlists/rockyou.txt -t 64 -vV 192.168.57.3 ssh
```

![image](https://user-images.githubusercontent.com/83584144/210282818-832f0b7b-4731-433c-92ae-204f908a86bb.png)

Conseguimos assim as credenciais do Jan vamos então realizar uma conexão ssh.

![image](https://user-images.githubusercontent.com/83584144/210282821-1cba690e-e2ab-4a78-b3aa-14170e41c44a.png)

## Escalada de privilégios:

Temos a sessão ssh no user jan, a partir daqui podemos começar a analisar os diretórios da máquina e as permissões dos user em questão.

Ao fazer 'ls' não encontramos nada do diretório.
```bash
Pwd
```

![image](https://user-images.githubusercontent.com/83584144/210282830-7b7ce134-4f76-44e6-af5c-2ed8a36fd0f6.png)

Vamos voltar para trás e analisar o diretório de outros users (neste caso só temos o Kay).

![image](https://user-images.githubusercontent.com/83584144/210282837-0e10473b-ea07-45fa-845e-201c64939b0d.png)

Como podemos ver neste diretório já temos um ficheiro que pelo nome parece interessante.

Tentei fazer 'nano' ao ficheiro e…

![image](https://user-images.githubusercontent.com/83584144/210282841-e160f92b-1962-45fb-a509-7c97ee048f61.png)

O jan não tem permissões para visualizar este ficheiro.

Tentei o 'ls -a' para ver se tinha mais informação:

![image](https://user-images.githubusercontent.com/83584144/210282848-ed777829-2b82-4894-97d4-d8a7239a0287.png)

Podemos ver que o user kay tem permissões de admin e por acaso vi que tinha o '.viminfo' o que me fez tentar abrir o ficheiro com o 'vim'.

![image](https://user-images.githubusercontent.com/83584144/210282854-7541324d-da67-4cf5-bac1-a7e82fe8da09.png)

E consegui abrir o ficheiro.

Muito bem temos um ficheiro chamado 'pass' no user Kay, então tentei realizar uma conexão ssh ao user Kay com a password encontrada.

![image](https://user-images.githubusercontent.com/83584144/210282859-51d8f332-1b65-410f-9905-25e30ae06390.png)

Resultou, entramos como user kay, fazemos o comando 'sudo su', voltamos a colocar a password e como podemos ver temos acesso root à máquina.

![image](https://user-images.githubusercontent.com/83584144/210282863-f7da40e0-0ba2-49b2-8a71-db136bedd023.png)

Aqui temos a flag.

