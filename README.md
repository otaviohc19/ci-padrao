# ci-padrao

Pipeline de verificação reutilizável para meus projetos JavaScript/Node no GitHub Actions.

| Job | O que faz | Reprova quando |
|---|---|---|
| Segredos (Gitleaks) | Varre **todo o histórico** do Git atrás de chaves, tokens e senhas | Encontra um segredo |
| Instalação, lint, testes e build | `npm ci`, `lint` e `test` (se existirem no package.json) e `build` | Algum passo falha |
| Dependências (npm audit) | Vulnerabilidades conhecidas nas dependências de produção | Há falha de nível `high` ou maior |
| Análise estática (Semgrep) | Regras de JavaScript, React, segredos e OWASP Top 10 | Achado de severidade WARNING ou ERROR |
| Lighthouse *(opcional)* | Acessibilidade, boas práticas, SEO e performance do build | Acessibilidade ou boas práticas abaixo de 90 |

## Como usar num projeto

1. Copie `modelos/ci.yml` para `.github/workflows/ci.yml`.
2. Copie `modelos/dependabot.yml` para `.github/dependabot.yml`.
3. Ajuste os `with:` se precisar (tabela abaixo).

| Entrada | Padrão | Para quê |
|---|---|---|
| `node-version` | `22` | Versão do Node |
| `diretorio` | `.` | Pasta do `package.json` (monorepo) |
| `build` | `true` | Rodar `npm run build` |
| `build-env` | vazio | Linhas `CHAVE=valor` gravadas no `.env` antes do build. **Só valores públicos**, vindos de *Variables* |
| `nivel-auditoria` | `high` | Nível mínimo que reprova no `npm audit` |
| `sast` | `true` | Semgrep (funciona em repositório privado) |
| `lighthouse` | `false` | Lighthouse na pasta do build (sites estáticos) |
| `pasta-build` | `dist` | Pasta gerada pelo build |

## CodeQL (só repositórios públicos)

Fica num workflow à parte, `codeql.yml`, porque precisa de permissão de escrita (`security-events: write`). O GitHub confere as permissões de todos os jobs, até dos desligados; se o CodeQL ficasse junto, todo projeto teria que liberar escrita, inclusive os privados. Veja o exemplo comentado em `modelos/ci.yml`.

## Decisões de segurança

- **Actions fixadas por SHA de commit**, com a versão no comentário. Uma tag pode ser movida; um SHA, não.
- **Ferramentas com versão fixa** (Gitleaks com checksum SHA-256, Semgrep por versão exata) e publicadas há pelo menos 2 semanas.
- **Permissões mínimas:** a verificação padrão pede só `contents: read`. O CodeQL, que precisa de escrita, fica em workflow separado.
- **`persist-credentials: false`** no checkout: o token não fica salvo no disco do runner.
- **`npm ci --ignore-scripts`:** instala exatamente o lockfile, sem rodar scripts de instalação de pacotes.
- **Dependabot com `cooldown` de 14 dias**, pela mesma regra das 2 semanas.

## Versões

Depois de mudar o workflow, crie uma tag nova (`v1.1.0`) e mova a `v1`:

```bash
git tag -a v1.1.0 -m "..." && git tag -f v1 && git push origin v1.1.0 && git push -f origin v1
```

Projetos que usam `@v1` recebem a mudança; para travar numa versão exata, use `@v1.1.0`.
