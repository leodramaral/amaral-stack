---
build:
  render: never
  list: never
  publishResources: false
---

# Plano de Desenvolvimento - Amaral Stack

## Observações sobre o tema enervoid

O tema já oferece de fábrica: mermaid, syntax highlighting (monokai), tags, layout responsivo, Swup (SPA transitions), KaTeX. **Precisaremos sobrescrever/complementar**: dark/light mode, i18n completo, paleta de cores, home page, branding, compartilhamento social, posts relacionados, DBML e Open Graph.

---

## Fase 1 - Fundação do Projeto

- `git init`

- `hugo new site` (ou estrutura manual)
- Tema enervoid como submodule
- `.gitignore` (incluindo `dev-flow.txt`)
- Criação do `dev-flow.txt`
- `hugo.toml` básico
- **Entregável**: site serve com `hugo server`, mostra o tema padrão
- **Commit**: `chore: inicialização do projeto Hugo com tema enervoid`

## Fase 2 - Estrutura i18n (BR/EN)

- Configurar `defaultContentLanguage = "br"` + `en` no `hugo.toml`
- Criar arquivos de tradução `i18n/br.toml` e `i18n/en.toml`
- Reestruturar `content/` para `content/br/` e `content/en/`
- Adicionar language switcher no header (override do partial)
- **Entregável**: site funcional nos dois idiomas com switcher
- **Commit**: `feat: suporte a multilinguagem BR/EN`

## Fase 3 - Branding (Logo, Favicon, Home, Redes Sociais)

- Favicon SVG com `<A/>`
- Navbar customizada com `<amaral stack/>` (override do `header.html`)
- Home page customizada: foto circular `perfil.jpg` + "Leandro Amaral" (override do `home.html`)
- Redes sociais no footer: GitHub, LinkedIn, X/Twitter
- **Entregável**: home page com foto circular e nome, navbar e footer corretos
- **Commit**: `feat: branding - logo, favicon, home page e redes sociais`

## Fase 4 - Paleta de Cores

- Override das cores do tema: substituir indigo por azul + verde vibrante
- Personalizar links, bordas, acentos, hovers, glow effects
- **Entregável**: site com identidade visual azul/verde/preto
- **Commit**: `feat: paleta de cores personalizada azul e verde vibrante`

## Fase 5 - Toggle Claro/Escuro

- Botão de toggle no header (sun/moon icon)
- CSS para tema claro (backgrounds, textos, bordas)
- Persistência via `localStorage`
- Mermaid re-renderiza com tema correto
- **Entregável**: toggle funcional com transição suave
- **Commit**: `feat: toggle de tema claro e escuro`

## Fase 6 - Features Sociais do Blog

- Botões de compartilhar nos posts (X, LinkedIn, copiar link)
- Seção de "posts relacionados" no final de cada post (baseado em tags)
- **Entregável**: posts com compartilhamento e sugestões de leitura
- **Commit**: `feat: compartilhamento social e posts relacionados`

## Fase 7 - Suporte Avançado de Conteúdo

- Render hook para blocos `dbml` (exibição formatada + conversão para mermaid erDiagram)
- Meta tags Open Graph (imagem, título, descrição por post)
- Verificar/ajustar syntax highlighting
- **Entregável**: posts suportam código, mermaid, DBML e Open Graph
- **Commit**: `feat: suporte a DBML, Open Graph e code blocks`

## Fase 8 - CI/CD (GitHub Pages)

- Workflow GitHub Actions para build e deploy
- Configuração de `baseURL` dinâmico
- **Entregável**: deploy automático ao push na main
- **Commit**: `ci: workflow de deploy via GitHub Pages`

## Fase 9 - Primeiro Post

- Conversão do `dev-flow.txt` em post narrativo (tom descritivo, informal, não step-by-step)
- Publicação nos dois idiomas (PT e EN)
- **Entregável**: primeiro post live no blog
- **Commit**: `feat: primeiro post - como este blog foi construído`

---

## Pontos de atenção

- **DBML**: Não existe renderizador JS amplamente disponível via CDN como o Mermaid. A proposta é renderizar DBML como bloco formatado + permitir que o autor use `erDiagram` do Mermaid para a visualização gráfica quando desejar.
- Cada fase gera **um commit** e só avança após aprovação manual.
- O `dev-flow.txt` será alimentado ao longo de todo o processo.

## Decisões tomadas

- **Redes sociais**: GitHub, LinkedIn, X/Twitter (footer + compartilhamento)
- **i18n**: Conteúdo + UI nos dois idiomas
- **Primeiro post**: Nos dois idiomas (PT e EN)
- **Repositório**: Será criado ao final; commits feitos localmente por enquanto
