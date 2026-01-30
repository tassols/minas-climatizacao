# Sistema de Metas, Pontos e Bônus — Minas Climatização

Sistema interno para gestão de metas, pontos e bônus por carro/equipe: produção diária, regras de qualidade, PMOC e resultados mensais.

## Funcionalidades

- **Equipes**: cadastro de carros (técnico + auxiliar).
- **Produção**: registro diário (instalação = 3 pts, limpeza = 1 pt). Checklist (c/ fotos) só em **instalações (visitas)**; limpeza não tem. 1 por visita; não fazer = −1 pt cada. Campos: instalações, limpezas, checklists feitos.
- **Qualidade**: apenas retorno Tipo A por carro/mês (retorno → perda do bônus operacional).
- **PMOC**: contratos ativos por técnico (R$ 40/mês) e registro de falha técnica.
- **Resultados**: página separada para exibir na TV, com atualização automática a cada 2 minutos.

## Como rodar localmente

1. Clone o repositório ou baixe os arquivos.
2. Use um **servidor local** (com Supabase, não abra `index.html` direto como arquivo — use sempre HTTP):
   ```bash
   npx serve -p 3000
   ```
   e acesse `http://localhost:3000`.

## Subir ao vivo (GitHub Pages)

1. Crie um repositório no GitHub.
2. Envie os arquivos na **raiz** do repo (`logo.png`, `index.html`, `config.js`, etc.). Logo como `logo.png` na raiz; `assets` opcional.
3. **Settings** → **Pages**. Source: **Deploy from a branch**. Branch: `main` (ou `master`), pasta **/ (root)** → **Save**.
4. Em alguns min o site fica em `https://<usuario>.github.io/<repo>/`. Ex.: **https://tassols.github.io/minas-metas/** e **.../minas-metas/resultados.html** (TV).
5. Para atualizar: **Add file** → **Upload files** no GitHub, ou `git add .` → `git commit -m "Atualizar"` → `git push origin main`.

- Site em **HTTPS**. Com **Supabase** em `config.js`, dados na nuvem; sem isso, só **localStorage**.

## Persistência dos dados (evitar perda)

Sem configuração extra, os dados são salvos só no **localStorage** do navegador. Se limpar o cache ou usar outro dispositivo, os dados se perdem.

Para **persistência na nuvem** (dados não se perdem):

1. Crie um projeto em [Supabase](https://supabase.com) (plano gratuito).
2. No **SQL Editor**, execute:

   ```sql
   create table if not exists minas_data (
     id text primary key default 'default',
     payload jsonb not null,
     updated_at timestamptz default now()
   );

   alter table minas_data enable row level security;

   create policy "Allow anonymous read/write for minas_data"
     on minas_data for all
     using (true)
     with check (true);
   ```

3. Em **Project Settings → API**, copie a **Project URL** e a **Publishable API Key** (ou "anon public").
4. No projeto, abra `config.js` e preencha `url` e `anonKey` com esses valores.
5. Salve, recarregue o site (ou refaça o deploy). Os dados passam a ser gravados no Supabase.

Se não for usar Supabase, não altere `config.js`. O sistema usará apenas o **localStorage** (use **Exportar backup** com frequência).

Enquanto salva, o sistema mostra **"Salvando…"** e depois **"Sincronizado às HH:MM"** (ou **"Salvo às HH:MM"** se estiver só em localStorage). Use **Exportar backup** com frequência como cópia extra.

---

## Checklist de configuração

Use esta lista para conferir se está tudo certo:

- [ ] **Supabase**
  - [ ] Projeto criado em [supabase.com](https://supabase.com).
  - [ ] Tabela `minas_data` criada (SQL acima no SQL Editor).
  - [ ] RLS ativado e política criada (incluída no SQL).
- [ ] **config.js**
  - [ ] `url`: Project URL do Supabase (ex.: `https://xxxx.supabase.co`).
  - [ ] `anonKey`: **Publishable API Key** ou **anon public** (JWT longa) do **Project Settings → API**.
  - [ ] Chave **idêntica** ao painel (sem espaço, nada a mais/menos). Erro *"Invalid API key"* ou *"Salvo localmente (erro ao sincronizar)"* → troque pela chave **anon public** (Legacy API Keys) se a publishable falhar.
- [ ] **Teste**
  - [ ] Abrir `index.html` (ou o site no GitHub Pages).
  - [ ] Cadastrar uma equipe e salvar.
  - [ ] No rodapé, deve aparecer **"Sincronizado às HH:MM"** (com Supabase) ou **"Salvo às HH:MM"** (só localStorage).
- [ ] **Backup**
  - [ ] Clicar em **Exportar backup** e guardar o JSON em local seguro. Fazer isso com frequência.
- [ ] **TV**
  - [ ] Abrir `resultados.html` (ou `.../resultados.html` no deploy).
  - [ ] Escolher mês/ano e conferir se os resultados batem com o sistema.
  - [ ] Na TV, deixar essa página aberta; ela atualiza sozinha a cada 2 minutos.

## Logo

A logo fica como `logo.png` na **raiz** do projeto. Pode usar `assets/logo.png` se preferir; nesse caso, ajuste os `src` da logo em `index.html` e `resultados.html`.

## Página de resultados (TV)

- **URL**: `resultados.html` (no mesmo domínio do deploy, ex.: `https://...github.io/.../resultados.html`).
- Use essa página na TV da empresa para todos acompanharem.
- Na TV é possível apenas trocar o mês/ano; **não há link para o sistema admin**, para evitar que colaboradores façam alterações a partir da tela da TV.
- A página recarrega os dados a cada 2 minutos.

## Estrutura dos arquivos

```
minas-metas/
├── index.html       # Sistema principal (equipes, produção, qualidade, PMOC)
├── resultados.html  # Página para TV
├── logo.png         # Logo (na raiz)
├── style.css
├── style-tv.css
├── app.js
├── resultados.js
├── shared.js
├── db.js
├── config.js
├── config.example.js
└── README.md
```

## Metas oficiais (referência)

| Nível | Pontos/dia | Pontos/mês | Bônus  |
|-------|------------|------------|--------|
| 1     | 6          | 132        | —      |
| 2     | 7–8        | 154–176    | R$ 400 |
| 3     | 9–10       | 198–220    | R$ 800 |
| 4     | 11+        | 242+       | R$ 1.200 |

- Bônus operacional: 60% técnico, 40% auxiliar.
- Bônus qualidade: R$ 200 (zero retorno Tipo A).
- Bônus eficiência: R$ 200 (média ≥ 9 pts/dia).
- Checklist (com evidência fotográfica) em Produção: só para **instalações (visitas)**; 1 por visita. Limpeza não tem. Cada um não feito = −1 pt.
- PMOC: R$ 40/mês por contrato ativo; falha técnica remove o bônus PMOC do mês.

---

### Erro "Invalid API key" ou 401 ao sincronizar

1. Abra **Project Settings → API** no Supabase.
2. Confira a **Project URL** e compare com `url` no `config.js` (ex.: `gqorlcpsztktyrelmjgw` — sem trocar `gq` por `gg` etc.).
3. Em **API Keys**, teste primeiro a **anon public** (aba "Legacy API Keys" ou "anon" / "public"). É uma chave longa em formato JWT. Copie e cole em `anonKey` no `config.js`.
4. Se o projeto só tiver **Publishable API Key**, copie de novo (sem cortar). Não pode ter espaço no início/fim.
5. Salve o `config.js`, recarregue a página e cadastre de novo. O rodapé deve mostrar **"Sincronizado às HH:MM"**.

---

**Documento interno — Uso restrito. Minas Climatização.**
