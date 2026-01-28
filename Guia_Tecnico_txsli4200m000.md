# Guia Técnico — txsli4200m000

## Identificação do Script

- **Programa:** txsli4200m000  
- **Descrição:** Invoicing Position (Exportação de notas de saída)  
- **Extensão:** B61O_a_ext  
- **Script Type:** 123  
- **Autor original:** Clauber Correia  
- **Data de criação:** 28/10/2021  

---

## Objetivo Técnico

Este script tem como finalidade **extrair e consolidar informações de faturamento (saídas)** do Infor LN, aplicando filtros operacionais e fiscais, e **gerar um arquivo CSV** com dados financeiros, tributários e cadastrais das notas fiscais emitidas.

O arquivo gerado é automaticamente disponibilizado para download ao final da execução.

---

## Estrutura Geral de Execução

Fluxo principal:

1. Inicialização de filtros de tela  
2. Criação do arquivo CSV (`saidas.csv`)  
3. Leitura das notas fiscais de saída (`btsli200`)  
4. Validação do parceiro de negócios  
5. Cálculo de impostos  
6. Consolidação financeira  
7. Escrita do registro no CSV  
8. Download e limpeza do arquivo temporário  

---

## Tabelas Utilizadas

| Tabela | Descrição |
|------|---------|
| `btsli200` | Cabeçalho da nota fiscal de saída |
| `btsli201` | Linhas financeiras / contábeis |
| `btsli202` | Impostos da nota |
| `btsli203` | Detalhamento complementar |
| `lpbra240` | Documento financeiro |
| `lpbra241` | Ordem relacionada |
| `btftb002` | Dados fiscais do parceiro |
| `btcom100` | Parceiro de negócios |
| `btcom130` | Endereço fiscal |
| `tcmcs003` | Armazém |
| `ttcmcs003` | Cache / domínio auxiliar |

---

## Parâmetros de Entrada (Tela)

Os filtros são tratados em pares **From / To**, com propagação automática:

- Referência (`frm.fire.f` → `frm.fire.t`)
- Data (`frm.date.f` → `frm.date.t`)
- CFOP (`frm.ccfo.f` → `frm.ccfo.t`)
- Tipo de Documento (`frm.fdtc.f` → `frm.fdtc.t`)
- Parceiro (`frm.bpid.f` → `frm.bpid.t`)
- Status da NF (`frm.stat.f` → `frm.stat.t`)
- Situação estadual (`frm.cste.f` → `frm.cste.t`)
- Projeto (`frm.cprj.f` → `frm.cprj.t`)

---

## Geração do Arquivo CSV

- **Caminho:** `${BSE}/appdata/relatorio/saidas.csv`
- **Separador:** `;`
- **Codificação:** padrão LN

### Cabeçalho do CSV

```
Referencia;Nota Fiscal;Serie;COD.PN;Cliente;CNPJ;Status;Status NFE;Origem;Fatura;
Data Geracao;Data Fiscal;CFOP;Grupo Contabil;Tipo NF;
Base ICMS;Valor ICMS;Base IPI;Valor IPI;Base PIS;Valor PIS;
Base Cofins;Valor Cofins;Base ICMS ST;Valor ICMS ST;
Valor Mercadoria;Seguro;Frete;Outras Despesas;Juros;Desconto;Valor Total;
Inscricao Estadual;Suframa;Chave Acesso;Contribuinte ICMS
```

---

## Função Principal — `read.main.table()`

Responsabilidades:

- Criação do arquivo CSV
- Leitura de `btsli200` com filtros
- Conversão e formatação de datas
- Validação fiscal do parceiro (`validate_bipd`)
- Leitura e consolidação de impostos
- Busca de ordens e documentos financeiros
- Escrita final do registro

Controle de fluxo é feito via `continue` quando a validação fiscal falha.

---

## Validação de Parceiro — `validate_bipd()`

Valida se o parceiro:

- Possui endereço fiscal válido (`btcom130`)
- Atende ao intervalo de **situação estadual (CSTE)** informado

Retornos:

- `0` → parceiro válido  
- `1` → parceiro inválido  

---

## Cálculo de Impostos

Impostos tratados a partir de `btsli202`:

- ICMS
- IPI
- PIS
- COFINS
- ICMS ST

Valores separados em:

- **Base de cálculo (`bc.*`)**
- **Valor do imposto (`vl.*`)**

---

## Juros e Descontos

- Descontos são identificados quando valores são negativos
- Juros quando positivos
- Consolidado a partir de `btsli201`

---

## Função `insert.registro()`

Responsável por:

- Montar a linha CSV
- Formatar valores monetários (`edit$`)
- Inserir o registro no arquivo

Nenhuma regra de negócio é aplicada aqui — apenas persistência.

---

## Limpeza de Variáveis — `clear.variables()`

Executada a cada iteração para evitar:

- Acúmulo de valores
- Vazamento de dados entre notas

Inclui limpeza de impostos, datas, descontos e variáveis auxiliares.

---

## Pontos de Atenção

- Dependência direta da estrutura fiscal brasileira (CFOP, ICMS, IPI, etc.)
- Alterações em impostos exigem ajuste em múltiplas funções
- Layout CSV é **contrato implícito** com processos externos
- Não há controle de concorrência para geração do arquivo

---

## Observações de Manutenção

- Alterar layout do CSV exige ajuste no cabeçalho e em `insert.registro`
- Filtros de tela são propagados manualmente
- Ideal manter versionamento externo (Git)

---

**Fim do Guia Técnico**
