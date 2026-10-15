# Certificados de calibração — MVP (HTML, CSS e JavaScript)

Projeto Aplicado III · UniSENAI · Empresa parceira: Schulz S.A. · AV 03 (Parte 1)

Aplicação que roda inteira no navegador, sem servidor e sem bibliotecas externas: o usuário envia
uma planilha `.xlsx`, o sistema calcula média, erro sistemático e conformidade por ponto, e
exibe o parecer final (`APROVADO` / `REPROVADO`).

## Como usar

Abra o `index.html` no Chrome, Edge ou Firefox (duplo clique) e envie uma planilha.
Há exemplos em `exemplos/`.

## Estrutura

```
index.html                 página (só estrutura)
css/style.css              estilos (tokens de cor, layout, modo escuro, impressão)
js/
  erros.js                 ErroPlanilha (erro com mensagem para o usuário)
  calculos.js              regras metrológicas — funções puras
  leitor-xlsx.js           abre o .xlsx (ZIP + XML) e devolve as linhas
  leitor-planilha.js       valida o layout e converte linhas em pontos de ensaio
  interface.js             monta a tela (resultado, tabela, erros)
  main.js                  liga tudo: arquivo -> leitura -> cálculo -> tela
tests/                     testes automatizados (Node)
exemplos/                  planilhas de exemplo (aprovada e reprovada)
docs/evidencias/           capturas de tela da aplicação
```

Cada módulo tem uma única responsabilidade e depende só dos anteriores na ordem de carregamento do
`index.html`. As regras de negócio (`calculos.js`) não conhecem DOM nem arquivos, por isso são
testáveis isoladamente.

## Regras de negócio

1. **Média** = soma das leituras ÷ quantidade de leituras do ponto.
2. **Erro sistemático** = média − valor padrão.
3. **Validação por ponto**: `DENTRO_DA_TOLERANCIA` se |erro| ≤ LEA, senão `FORA_DA_TOLERANCIA`.
4. **Parecer final**: `APROVADO` se todos os pontos estão dentro; `REPROVADO` se algum estiver fora.

Como JavaScript usa ponto flutuante, a comparação com o LEA tem uma tolerância de 1e-9
(`EPSILON` em `calculos.js`); assim, um erro exatamente igual ao LEA é conforme.

## Layout da planilha

Primeira aba, linha 1 = cabeçalho, uma linha por ponto:

| Ponto | Valor_Padrao | LEA | Leitura_1 | Leitura_2 | … | Leitura_n |
|---|---|---|---|---|---|---|
| P1 | 2,00 | 0,05 | 2,01 | 2,02 | … | 2,01 |

Aceita variações de cabeçalho (acentos, maiúsculas, `Leitura 1`), quantidade variável de leituras,
células vazias e vírgula ou ponto decimal.

## Testes

Requer Node 18+ (só para os testes; a aplicação não precisa de Node):

```bash
node --test tests/*.test.js
```

## Premissas e limites

- O roteiro não define o layout da planilha da Schulz; o acima é uma proposta. Se a real for
  diferente, adapte só `js/leitor-planilha.js`.
- Lê apenas `.xlsx` (não `.xls` nem `.csv`), tamanho máximo de 5 MB.
- Usa `DecompressionStream`, presente nos navegadores atuais (Chrome/Edge 80+, Firefox 113+, Safari 16.4+).
- Os dados de exemplo são fictícios.

## Publicar no GitHub Pages (opcional)

No repositório: Settings → Pages → Deploy from a branch → `main` / `(root)`.
