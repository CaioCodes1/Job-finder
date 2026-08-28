projeito criado para aplicar habilidades mais avançadas de javascript usando, html, css, java script, handlebars, usando um pouco de banco de dados para aplicações futuras mais avançadas com,
e assim fazendo um site responsivo conseguindo aplicar o que foi aprendido até agora no curso de java script do basico ao avançado

## Dependências e segurança

```bash
npm install
npm audit
npm run dev    # sobe na porta 3000
```

Uma auditoria em 28/08/2026 encontrou **2 falhas críticas e 5 altas**. A mais séria era no `handlebars` — **injeção de JavaScript** por confusão de tipo na AST — e é justamente o motor de template que este projeto usa para renderizar todas as páginas. Junto vinham `tar` (crítica), `lodash`, `path-to-regexp`, `picomatch`, `brace-expansion` e `ip-address`.

Todas saíram com `npm audit fix`, sem quebra de compatibilidade.

### O `overrides` do uuid

Sobraram duas moderadas na cadeia `sequelize` → `uuid@8.3.2`, e a correção que o `npm audit` propunha era **rebaixar o `sequelize` de 6.37.8 para 3.30.0** — três versões maiores para trás, o que quebraria o projeto inteiro para resolver uma falha moderada.

O `uuid` só é corrigido a partir da 11.1.1, e nenhuma versão do `sequelize` pede essa: a 6.37.8 já é a mais nova da linha 6, e não existe `sequelize` 7 estável. A saída foi forçar a versão corrigida por baixo:

```json
"overrides": {
  "uuid": "^11.1.1"
}
```

Pular três versões maiores por baixo de uma biblioteca é exatamente onde as coisas quebram calado, então a troca só ficou de pé depois de verificada:

| Verificação | Resultado |
|---|---|
| `db.authenticate()` e `db.sync()` | conecta e valida a tabela |
| `Job.create` / `findByPk` / `findAll` / `destroy` | as quatro operações funcionam |
| `GET /` e `GET /jobs/add` | HTTP 200, templates renderizando |
| `npm audit` | **0 vulnerabilidades** |

Quando o `sequelize` passar a exigir o `uuid@11` sozinho, o `overrides` deixa de ser necessário e deve sair.

### Sobre o histórico do repositório

O `node_modules` estava versionado — 5.155 arquivos, 64 MB num projeto cujo código cabe em 200 KB. Saiu do rastreamento junto com a criação do `.gitignore`, que não existia (era essa a causa).

Os arquivos continuam nos dois commits anteriores, então um `git clone` ainda baixa o volume antigo. Limpar isso de vez exige reescrever o histórico e um `push --force`.
