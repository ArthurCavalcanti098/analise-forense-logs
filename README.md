# Investigação Forense de Logs & Inteligência de Ameaças

Laboratório prático de análise forense em que investiguei o histórico de acessos (`access.log`) de um servidor Linux que sofreu varreduras automatizadas e um ataque direcionado. Usei terminal, CyberChef e VirusTotal para isolar requisições suspeitas, decodificar dados escondidos e rastrear a origem das conexões.

## O que foi feito

- Extração dos arquivos e filtragem do `access.log` com `grep` para isolar requisições anômalas
- Identificação de uma requisição com dado codificado em Base64 na URL (padrão `==`)
- Decodificação de artefatos no CyberChef
- Uso de regex para localizar um valor válido no meio de muito ruído
- Análise de origem e reputação de IPs suspeitos no VirusTotal

## Ferramentas

| Ferramenta | Uso |
|---|---|
| Linux (`grep`, `cat`, `unzip`) | Extração e filtragem dos logs |
| CyberChef | Decodificação de Base64 e uso de regex |
| VirusTotal | País de origem, provedor e reputação dos IPs |

## Relatório completo

O passo a passo detalhado, com os comandos usados, os resultados e as conclusões, está em [DesafioCompleto.pdf](./DesafioCompleto.pdf).

## Aprendizados

- Identificar se um dado é Base64 ou hexadecimal antes de escolher a receita no CyberChef. Seguir o roteiro cegamente pode não funcionar.
- Fazer triagem rápida de logs grandes usando padrões, como o `==` do Base64.
- Usar regex para achar um valor válido no meio de dados ruidosos.
- Dar contexto a um ataque com inteligência de ameaças (país, provedor e reputação do IP).

## Observação

Este repositório contém apenas a documentação do meu processo. Os arquivos do laboratório (logs e artefatos) não estão incluídos.

## Autor

**Arthur Cavalcanti**
