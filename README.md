# Investigação Forense de Logs & Inteligência de Ameaças

Laboratório prático de análise forense em que investiguei o histórico de acessos (`access.log`) de um servidor Linux que sofreu varreduras automatizadas e um ataque direcionado. Usei terminal, CyberChef e VirusTotal para isolar requisições suspeitas, decodificar dados escondidos e rastrear a origem das conexões.

**Participante:** Arthur Cavalcanti
**Data:** 28/09/2026
**Arquivos analisados:** `access.log` e `encodedflag.txt` (dentro do `loganalysis-1695809155170.zip`)
**Ferramentas:** Terminal (`grep` e `cat`), CyberChef e VirusTotal

---

## 1. Resumo

Neste laboratório eu analisei o histórico de acessos de um servidor Linux que sofreu varreduras automatizadas e um ataque direcionado. A ideia era achar a requisição suspeita, decodificar os dados escondidos e descobrir de onde vieram as conexões. Abaixo estão as respostas das quatro perguntas e, mais adiante, o passo a passo do que eu fiz.

| # | Pergunta | Resposta |
|---|----------|----------|
| 1 | IP do atacante que tentou extrair a flag Base64 no log | `212.14.17.145` |
| 2 | Frase secreta revelada com From Base64 | `THM{CYBERCHEF_WIZARD}` |
| 3 | País e empresa do IP 54.36.115.221 (varreu o `.env`) | França (FR), OVH |
| 4 | Mensagem descriptografada do `encodedflag.txt` | `08-2E-9A-4B-7F-61` |

---

## 2. Passo a passo

### 2.1 Preparando o ambiente e extraindo os logs

Criei uma pasta só para o laboratório e extraí o ZIP dentro dela:

```bash
cd ~/Documentos
mkdir projeto-analise
unzip loganalysis-1695809155170.zip -d projeto-analise
cd projeto-analise
```

### 2.2 Filtrando o log (Missão 1)

O `access.log` é bem grande e quase tudo nele é tráfego normal. Como o Base64 costuma terminar com `=` ou `==`, usei o `grep` para mostrar só as linhas com esse padrão:

```bash
grep "==" access.log
```

Apareceu a linha suspeita:

```
212.14.17.145 - - [27/Sep/2023:07:00:53 +0000] "GET /VEhNe0NZQkVSQ0hFRl9XSVpBUkR9== HTTP/1.1" 401 5196 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:101.0) Gecko/20100101 Firefox/101.0"
```

O que dá para tirar dessa linha:

- **IP de origem:** `212.14.17.145`.
- **Requisição:** um `GET` para um "arquivo" que na verdade é uma string em Base64 (`VEhNe0NZQkVSQ0hFRl9XSVpBUkR9==`).
- **Status 401:** o servidor exigiu autenticação e não liberou o acesso.
- **User-Agent:** finge ser um Firefox no Windows, provavelmente para parecer tráfego normal.

### 2.3 Decodificando a string da URL (Missão 2, parte 1)

Copiei a string sem a barra inicial e sem o `GET`, colei no Input do CyberChef e usei a operação **From Base64**. O Output foi:

```
THM{CYBERCHEF_WIZARD}
```

O Output também mostrou uns caracteres estranhos antes e depois da flag (`Dÿ` e `ÓLÿõ`). São bytes que não têm representação em texto, então ignorei. A flag é só o trecho `THM{CYBERCHEF_WIZARD}`.

### 2.4 Decodificando o encodedflag.txt (Missão 2, parte 2)

Aqui eu travei. Usei **From Hex** e depois **From Base64**, mas fazendo assim o Output vinha vazio. Depois de conversar com a coordenadora e analisar o arquivo, entendi o motivo: o conteúdo do `encodedflag.txt` é Base64 direto, com letras como M, T, y e z e terminando em `=`, que não existem em hexadecimal. Então o From Hex não tinha o que decodificar.

O caminho que funcionou foi:

1. Colar o conteúdo do arquivo no Input e aplicar só o **From Base64**.
2. O resultado é uma sequência longa de blocos hexadecimais separados por hífen, no formato de endereços MAC, com quase todos mal formados de propósito (por exemplo `08-2E-9A-4B-7FE-610A-3C-...`).
3. Adicionar a operação **Regular expression** e escolher o preset **MAC address** (equivalente a `([0-9A-Fa-f]{2}-){5}[0-9A-Fa-f]{2}`) para achar o único valor válido no meio do ruído.

O único MAC válido encontrado foi:

```
08-2E-9A-4B-7F-61
```

### 2.5 Investigando os IPs no VirusTotal (Missão 3)

Pesquisei o IP `54.36.115.221`, que aparece no log tentando ler o arquivo `.env` (onde ficam variáveis de ambiente e credenciais). Nas abas *Detection* e *Details* do VirusTotal:

| Campo | Resultado |
|-------|-----------|
| Continente | Europa (EU) |
| País | França (FR) |
| Empresa (whois RIPE) | OVH (org-name: OVH GmbH, ORG-OG9-RIPE) |
| Tipo de organização | OTHER |
| Reputação | -4 |

A reputação negativa mostra que a comunidade de segurança já marcou esse IP como malicioso, o que combina com varredura automatizada em busca de arquivos sensíveis. A OVH é uma empresa de hospedagem e cloud, muito usada por scanners e bots, mas o IP estar nela não quer dizer que a própria OVH seja responsável pelo ataque.

---

## 3. Conclusões

- O servidor foi alvo de varredura automatizada (o IP `54.36.115.221` tentando ler o `.env`) e de uma requisição suspeita com dado codificado em Base64 na URL (o IP `212.14.17.145`).
- Base64 é uma forma simples de esconder informação de filtros básicos. Procurar o padrão `==` no log é uma triagem rápida e eficiente.
- O ataque da flag foi negado com status 401, ou seja, não teve acesso liberado.
- Como melhorias, eu bloquearia os IPs identificados no firewall ou WAF, impediria o acesso público a arquivos como o `.env` e monitoraria requisições com strings Base64 em URLs.

---

## Observação

Este repositório contém apenas a documentação do meu processo. Os arquivos do laboratório (logs e artefatos) não estão incluídos.

## Autor

**Arthur Cavalcanti**
