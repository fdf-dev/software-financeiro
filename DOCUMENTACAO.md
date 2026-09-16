# Software Financeiro

Documentação funcional e técnica do estado atual da aplicação.

## 1. Visão geral

O Software Financeiro é uma aplicação web para planejamento financeiro de projetos. Ele permite configurar um projeto, cadastrar equipe, alunos, serviços e locomoção, consultar a contrapartida econômica e executar uma simulação de juros compostos.

A aplicação foi construída com Vue 3, Vite e Tailwind CSS. O estado atual é mantido em memória no navegador. Não há banco de dados, autenticação ou persistência automática em `localStorage`.

## 2. Como executar

Requisitos:

- Node.js e npm instalados.

Instalar dependências:

```bash
npm install
```

Iniciar o ambiente de desenvolvimento:

```bash
npm run dev
```

Gerar a versão de produção:

```bash
npm run build
```

Visualizar a versão de produção localmente:

```bash
npm run preview
```

## 3. Fluxo da aplicação

O fluxo principal possui as seguintes etapas:

1. Projeto
2. Equipe
3. Alunos
4. Serviços e locomoção
5. Contrapartida Econômica
6. Simulação

As etapas aparecem em sequência. Depois que o projeto é configurado, etapas anteriores podem ser reabertas. As etapas futuras são liberadas conforme as validações das etapas anteriores são atendidas.

## 4. Etapa Projeto

Permite informar:

- Nome do projeto, que é opcional.
- Tempo de desenvolvimento em meses.
- Data de início prevista.
- Se existe garantia e encerramento.
- Quantidade de meses de garantia e encerramento.

O sistema calcula e exibe:

- Último mês de desenvolvimento.
- Data de garantia e encerramento, quando a garantia está habilitada.

Validações principais:

- O tempo de desenvolvimento deve ser maior que zero.
- A data de início é obrigatória.
- Os meses de garantia são obrigatórios e devem ser maiores que zero quando a opção estiver marcada.

Também existem ações para:

- Importar um cadastro completo em CSV.
- Limpar somente a página atual.
- Limpar os dados do projeto.

## 5. Etapa Equipe

Cada membro da equipe possui os seguintes campos:

- Titulação.
- CPF.
- Nome.
- Salário.
- Meses de participação.
- Início da participação.
- Fim da participação, calculado automaticamente.
- Horas semanais.

As titulações disponíveis são Técnico, Graduação, Especialização, Mestrado e Doutorado.

Funcionalidades:

- Adicionar membros.
- Remover membros adicionais.
- Definir o período de participação.
- Calcular automaticamente o fim da participação.
- Selecionar o mês de início dentro do período permitido.

Validações:

- Titulação, CPF, nome, salário, período e horas semanais devem ser preenchidos.
- O CPF deve possuir 11 dígitos e passar na validação dos dígitos verificadores.
- Os meses de participação devem ser maiores que zero e não podem exceder o tempo de desenvolvimento.
- O início da participação deve estar dentro do período do projeto.
- O período completo de participação deve caber no projeto.

O primeiro membro é exibido como uma linha inicial vazia. Novas linhas são adicionadas com horas semanais padrão de 40.

## 6. Etapa Alunos

Cada aluno possui:

- Titulação.
- Nome.
- Meses de participação.
- Início da participação.
- Fim da participação, calculado automaticamente.
- Horas semanais.
- Valor da bolsa.

Funcionalidades:

- Adicionar alunos.
- Remover alunos adicionais.
- Definir início e duração da participação.
- Calcular automaticamente o fim da participação.

Regras:

- O primeiro aluno é criado como `Aluno 1`.
- O limite de meses do aluno é o tempo de desenvolvimento menos dois meses.
- Titulação, nome, duração, início e horas semanais são considerados na validação da etapa.
- O valor da bolsa é armazenado no cadastro, mas não participa da simulação de juros atual.

## 7. Etapa Serviços e locomoção

Cada item possui:

- Tipo.
- Descrição.
- Unidade.
- Valor.

Tipos previstos:

- Diárias.
- Passagens aéreas.
- Serviços de terceiros.
- Assinaturas.
- Compra de bens físicos.

Funcionalidades:

- Adicionar itens.
- Remover itens adicionais.
- Importar orçamento por CSV.
- Baixar modelo de orçamento.

Validações:

- Tipo e descrição são obrigatórios.
- Unidade deve ser maior ou igual a 1.
- Valor deve ser informado e não pode ser negativo.

Os itens cadastrados nesta etapa ainda não são somados à simulação de juros.

## 8. Etapa Contrapartida Econômica

A contrapartida exibe os membros importados da etapa Equipe e permite cadastrar pessoas exclusivamente nesta etapa.

Campos exibidos:

- Nome da equipe.
- Salário.
- Meses de participação.
- Início da participação.
- Horas mensais.

Regras de edição:

- Nomes e salários importados da aba Equipe ficam bloqueados nesta etapa.
- Membros adicionados manualmente na Contrapartida podem ter nome e salário alterados.
- Membros adicionados manualmente na Contrapartida não são incluídos na aba Equipe.
- O cadastro manual da Contrapartida é mantido separado da lista de equipe.

Horas:

- O valor padrão de horas mensais é 40.
- Abaixo do campo é exibida a conversão para horas semanais.
- O cálculo utilizado é `horas mensais / 4`.
- Exemplo: 40 horas mensais correspondem a 10 horas semanais.

Valor da hora:

- Para docente: `(salário * 13,5 / 12) / 160`.
- Para não docente: `(salário * 13 / 12) / 160`.

O checkbox `Não é docente` altera automaticamente a fórmula utilizada. Quando o salário não é válido ou não foi informado, o valor da hora é exibido como `—`.

## 9. Etapa Simulação

A simulação recebe:

- Aporte inicial.
- Aporte mensal.
- Taxa mensal de juros.
- Período em meses.

O período é preenchido automaticamente com o tempo de desenvolvimento do projeto.

O resultado apresenta:

- Total investido.
- Total de juros.
- Montante final.

O cálculo aplica juros sobre o saldo a cada mês e, em seguida, adiciona o aporte mensal. Os valores são formatados em reais brasileiros.

A simulação é independente dos valores cadastrados em equipe, alunos, serviços e contrapartida. Atualmente, esses custos não são consolidados no resultado da simulação.

## 10. Importação de cadastro completo

A tela Projeto aceita arquivos CSV com cadastro de projeto, pesquisadores, alunos e serviços.

O importador:

- Aceita separador `;` ou `,`.
- Remove BOM UTF-8 quando presente.
- Reconhece números no formato brasileiro.
- Importa registros identificados como `projeto`, `pesquisador`, `equipe`, `aluno` e `servico`.
- Preenche as coleções correspondentes da aplicação.
- Exibe uma mensagem com a quantidade de registros importados.

As colunas principais do modelo são:

```text
registro;nome;titulacao;cpf;salario;horas semanais;meses participacao;inicio participacao;valor bolsa;tempo desenvolvimento;inicio projeto;tem garantia;meses garantia;tipo;descricao;unidade;valor
```

Registros de pessoas sem nome, titulação ou meses válidos são ignorados. O arquivo deve conter o cabeçalho do modelo para ser aceito.

## 11. Modelo de cadastro

A aplicação possui uma função para preencher dados de exemplo e baixar o arquivo `modelo-cadastro-completo.csv`, com:

- Um projeto de exemplo.
- Pesquisadores.
- Alunos.
- Um serviço de locomoção.

O arquivo é gerado no navegador, sem envio para servidor.

## 12. Limpeza de dados

As páginas possuem confirmação antes da limpeza.

É possível limpar:

- Projeto ou dados do projeto.
- Equipe financeira.
- Alunos.
- Serviços e locomoção.
- Dados da simulação.

A limpeza da equipe não deve ser confundida com a limpeza de membros adicionados exclusivamente na Contrapartida. Os dados manuais da Contrapartida são mantidos em uma coleção separada.

## 13. Estrutura técnica

Arquivos principais:

- `src/App.vue`: interface, estado, validações, cálculos, importação e navegação.
- `src/components/MonthPicker.vue`: componente reutilizável para seleção de mês e ano.
- `src/style.css`: estilos globais e configuração visual.
- `src/main.js`: inicialização da aplicação Vue.
- `public/modelo-cadastro-completo.csv`: arquivo CSV de referência.
- `package.json`: scripts e dependências do projeto.

Tecnologias utilizadas:

- Vue 3 com `script setup`.
- Vite.
- Tailwind CSS.
- `currency.js` para apoio à formatação e cálculos monetários.

## 14. Limitações atuais

- Os dados são perdidos ao recarregar ou fechar a página.
- Não há banco de dados, login, usuários ou sincronização.
- Não existe exportação do cadastro preenchido atual.
- Não existe um total financeiro consolidado da equipe, alunos e serviços.
- Serviços e bolsas não entram na simulação de juros.
- A contrapartida não apresenta um total econômico consolidado.
- A importação depende do cabeçalho e do formato do modelo CSV.
- A importação pode manter dados anteriores de categorias que não apareçam em um novo arquivo.
- Não há validação completa de todos os campos importados antes de liberar o uso.
- O cálculo de horas semanais da Contrapartida é uma referência baseada em quatro semanas por mês.

## 15. Estado de validação

O build atual da aplicação é executado com:

```bash
npm run build
```

Esse comando deve concluir com sucesso e gerar a pasta `dist/`.