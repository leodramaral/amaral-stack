# Amaral Stack

Blog pessoal bilíngue (pt-BR / en) e mini-CV de Leandro Amaral, construído com [Hugo](https://gohugo.io/) e o tema [enervoid](https://github.com/Enerhim/enervoid-theme). Sem pipeline de build de JS/CSS — não há `package.json`; Tailwind CSS v4, devicon e Font Awesome são carregados via CDN no `<head>`.

Publicado em GitHub Pages a partir de `main` via `.github/workflows/deploy.yml`.

## Pré-requisitos

- **Hugo Extended v0.146.0 ou superior.** O layout do site usa a convenção de diretórios `_partials` / `_markup`, introduzida nessa versão — um Hugo mais antigo (por exemplo, a versão empacotada pela distro Linux) falha ao renderizar com erros como `partial "head.html" not found`, mesmo com o arquivo existindo. Baixe a versão correta em [github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases) (procure pelo `.deb`/`.tar.gz` `extended`) e confirme com:

  ```sh
  hugo version
  # deve mostrar algo como: hugo v0.146.0+extended ...
  ```

O tema `enervoid` já vem versionado dentro do repositório em `themes/enervoid` — não é necessário nenhum passo extra de submódulo para tê-lo disponível após o clone.

## Rodando localmente

```sh
git clone <url-do-repo>
cd amaral-stack
hugo server
```

Isso sobe um servidor local com live reload (por padrão em `http://localhost:1313`).

## Build de produção

```sh
hugo --minify --baseURL "<url>/"
```

Gera os arquivos estáticos em `public/`. É o mesmo comando usado pelo workflow de deploy (`.github/workflows/deploy.yml`).

## Criando um novo post

Cada post existe como um par de arquivos, um em cada idioma, em diretórios espelhados dentro de `content/br/blog/` e `content/en/blog/`:

```sh
hugo new content br/blog/<slug>/index.md
hugo new content en/blog/<slug>/index.md
```

Os posts são "leaf bundles": o `index.md` e as imagens do post ficam juntos no mesmo diretório. O front matter inicial vem de `archetypes/default.md`.

## Estrutura do projeto

- `content/br/`, `content/en/` — conteúdo de cada idioma (posts de blog, páginas pessoais).
- `layouts/` — overrides de templates do tema `enervoid` (branding, i18n, TOC do blog, heading render hook, etc.). Hugo resolve primeiro os arquivos aqui, depois cai para `themes/enervoid/layouts/`.
- `layouts/i18n/br.toml` / `en.toml` — strings de UI traduzidas (fora do conteúdo dos posts).
- `assets/css/theme.css` — overrides de tema sobre as classes utilitárias do Tailwind, incluindo o modo claro.
- `themes/enervoid/` — o tema base, versionado diretamente no repositório.

## Notas

- Não há suíte de lint/test neste repositório.
- Detalhes mais profundos de arquitetura e convenções (padrão de override de layout, i18n, mermaid, etc.) estão documentados em `CLAUDE.md`, voltado para orientar agentes de IA trabalhando neste código.
