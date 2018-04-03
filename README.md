# Development Guidelines

## Sumário 

[INTRODUÇÃO](#introdução)

[Objetivo](#objetivo)

[CONCEITOS](#conceitos)

[Commits](#commits)

[Branches](#branches)

[Feature Branches](#feature-branches)

[WORKFLOW](#workflow)

[Situação: Quero desenvolver uma nova funcionalidade ou correção](#situação-quero-desenvolver-uma-nova-funcionalidade-ou-correção)

[Templates para Issues e Pull Requests](#templates-para-issues-e-pull-requests)


## INTRODUÇÃO

### DISCLAIMER
Esse guia já pressupõe que você saiba usar o Git e que você saiba como fazer um *commit*, *rebase* e *merge*.

### Objetivo
Não existe a forma certa de usar o git, assim como não existe a forma certa de se escrever um software ou um texto. Logo, esse guia não deve ser lido como a verdade, mas sim como um conselho amigo de uma forma mais conveniente e segura que achamos de usá-lo.

## CONCEITOS

### Commits

Por simplicidade, iremos dar 3 nomes aos commits:
- __*commits*__: commits feitos pelo usuário
- __*commits de merge*__: commits usando o comando `git merge <branch> --no-ff`
- __*commits de release*__: commits usando a ferramenta [releasy](https://www.npmjs.com/package/releasy)

O [releasy](https://www.npmjs.com/package/releasy) é uma ferramenta que facilita a mudança de versão do projeto. Ele cria um *commit* com a nova versão inserida no arquivo `package.json` e cria uma *tag* para esse *commit* com a sua versão. Iremos falar mais sobre ele a seguir.

As mensagens de *commit* devem ser escritas na lingua **inglesa** e seguir o seguinte modelo:
```
<Verbo no imperativo> <objeto da ação>
```

Exemplos:
- Add PayPal Plus as a new payment method
- Fix profile not showing
- Update React Router
- Remove unused dependency

O que fazer:
- Caso esteja fechando uma issue com o commit, coloque "Fix #<Número da Issue>" no corpo do commit, isto é:
```
<Verbo no imperativo> <objeto da ação>    <---- Titulo do commit
                                          <---- Linha em branco
Fix #<Número da issue>                    <---- Corpo do commit
```

O que *não* fazer:
- Colocar ponto no final da frase. Ex: "Update React Router."
- Iniciar com letra minúscula
- Escrever em português
- Referenciar número de issue no título

### Branches

Existem basicamente duas branches com nomes fixados: `master` e `beta`.

A branch `master` deve refletir exatamente o que está em produção, ou seja, ela deve ser tratada como __*the single source of truth*__. É dela que devem sair todas as branches de desenvolvimento.

Apesar da branch `beta` ter seu nome fixado, essa branch será a branch que mais será construída e descontruída, logo não guarde nada nela!

**Importante lembrar:** Apenas *commits de merge* devem ser feitos na branch `master` e `beta`.

#### Feature branches

Para começar uma nova funcionalidade ou correção, você deve abrir uma branch com base em `master`. Essa *branch* , apesar de nem sempre ser uma *feature*, é chamada de *feature branch*. Ela deve seguir o seguinte esquema de nome: `<type>/<description>`

Os possíveis `types` são:
- **feature:** nova funcionalidade ou comportamento
- **fix:** correção de bug
- **update:** atualização de dependência
 
A description deve ser curta, em inglês e em kebab-case. Ela deve dar uma noção básica do que está sendo desenvolvido nessa *branch*.

Ex: `git checkout -b feature/paypal-plus`.

**Importante lembrar:** Apenas *commits* deve ser feito em uma *feature branch*. Nenhum *commit de release ou de merge* deve ser feito em uma *feature branch*.

## WORKFLOW

### Situação: Quero desenvolver uma nova funcionalidade ou correção

**Passo 1.** Crie uma *feature branch* a partir de `master`.

```sh
git checkout master
git checkout -b feature/nice-new-thing
```

**Passo 2.** Desenvolva a nova funcionalidade nessa *branch* e faça os *commits* normalmente.

```sh
git commit -m "Add nice new thing"
```

**Passo 3.** Faça um *commit de merge* dessa *feature branch* em `beta` com a flag `--no-ff`.

```sh
git checkout beta
git merge feature/nice-new-thing --no-ff
```

**Passo 4.** Faça um *commit de release* na branch `beta` usando o releasy.

Caso seja a única *feature branch* em `beta`, use: `releasy`.
Caso já tenha outras *feature branches* em `beta`, use: `releasy pre`.

**Importante:** Certifique-se de que a versão publicada contenha o sufixo `-beta`.

**Passo 5.** Após publicar a nova versão, aguarde o build no Pachamama acabar e mude a versão publicada no Delorean.

**Passo 6.** Teste o trabalho feito no ambiente *beta* (vtexcommercebeta). Caso contenha bugs, vá para sua *feature branch* (`git checkout feature/new-nice-thing`) e volte para o passo 2. Caso contrário, continue para o passo 7.

**Passo 7.** Abra um Pull Request e marque um colega da equipe. PRs são nossos amigos, eles estão aí pra garantir que outra pessoa fique de olho no nosso código. Isso traz mais qualidade pra equipe e pro produto. É uma boa forma de aprender e de ter mais segurança ao colocar novas features em stable. Vá pro passo 8 caso alguém aprove o seu PR ou volte pro passo 2 pra corrigir algum problema.

**Passo 8.** Publicando uma *feature branch* em produção:

Faça um *commit de merge* dessa *feature branch* em `master` com a flag `--no-ff`.

```sh
git checkout master
git merge feature/nice-new-thing --no-ff
``` 

**Passo 9.** Faça um *commit de release* na branch `feature` usando o releasy.

- Incremente uma versão *minor* caso seja uma "feature" ou "update": `releasy minor --stable`
- Incremente uma versão *patch* caso seja "fix": `releasy patch --stable`

**Importante:** Verifique que a versão não contenha o sufixo `-beta`, caso tenha, você esqueceu de usar *flag* `--stable` no releasy.

**Passo 10.** Após publicar a nova versão, aguarde o build no Pachamama acabar e mude a versão publicada no Delorean.

**Passo 11.** Verifique as modificações em *stable* (vtexcommercestable) e acompanhe as métricas para verificar se a sua subida não causou nenhum dano. Caso algo esteja errado, volte a versão no Delorean **imediatamente**! Você poderá debugar o erro em *beta*.

**Passo 12.** Comemore! Você publicou algo em produção para milhões de pessoas!

### Situação: Atualizaram a branch `master` e eu estou desenvolvendo uma *feature branch*

Faça *rebase* da sua *feature branch*.

```sh
git checkout master
git pull
git checkout feature/nice-new-thing
git rebase master
git push origin feature/nice-new-thing -f
```

**Importante lembrar:** Sempre mantenha *feature branches* em desenvolvimento rebaseadas em `master`.

### Templates para Issues e Pull Requests 

Uma boa prática adotada por vários é utilizar templates na criação de Issues e PRs que ajudam a descrever o seu trabalho de maneira clara para todos. 

Para adicionar crie uma pasta `.github` na raiz do seu repositório e adicione os arquivos [PULL_REQUEST_TEMPLATE](https://github.com/vtex/dev-guidelines/blob/master/.github/PULL_REQUEST_TEMPLATE.md) e [ISSUE_TEMPLATE](https://github.com/vtex/dev-guidelines/blob/master/.github/ISSUE_TEMPLATE.md)

Agradecimentos aos projetos de **omnichannel** e **help-center** de onde esses templates foram pegos. 
