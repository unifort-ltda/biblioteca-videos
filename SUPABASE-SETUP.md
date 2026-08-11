# Publicação sem código — configuração única

1. Crie um projeto gratuito em https://supabase.com.
2. Em **SQL Editor**, execute todo o conteúdo abaixo.
3. Em **Authentication > Users**, crie o usuário que irá administrar a biblioteca.
4. Em **Project Settings > API**, copie a `Project URL` e a chave `anon public` para `supabase-config.js`.
5. Publique os arquivos no repositório `biblioteca-videos`. Acesse `admin.html`, faça login e clique uma vez em **Importar catálogo atual** para migrar os vídeos que já existem em `videos.json`.

```sql
create table public.videos (
  id uuid primary key default gen_random_uuid(),
  produto text not null,
  segmento text not null,
  categoria text not null,
  codigo text default '',
  descricao text default '',
  video_url text not null,
  thumbnail_url text default '',
  download_url text default '',
  published boolean not null default true,
  created_at timestamptz not null default now()
);

alter table public.videos enable row level security;

create policy "Vídeos publicados são públicos" on public.videos for select using (published = true);
create policy "Administradores veem todos os vídeos" on public.videos for select to authenticated using (true);
create policy "Administradores inserem vídeos" on public.videos for insert to authenticated with check (true);
create policy "Administradores atualizam vídeos" on public.videos for update to authenticated using (true) with check (true);
create policy "Administradores excluem vídeos" on public.videos for delete to authenticated using (true);
```

## Armazenamento de imagens e arquivos

Execute também este SQL no mesmo editor para que o painel envie capas e arquivos de download diretamente do computador:

```sql
insert into storage.buckets (id, name, public) values ('biblioteca-assets', 'biblioteca-assets', true);

create policy "Arquivos públicos da biblioteca" on storage.objects for select to anon, authenticated using (bucket_id = 'biblioteca-assets');
create policy "Administrador envia arquivos" on storage.objects for insert to authenticated with check (bucket_id = 'biblioteca-assets' and (auth.jwt() ->> 'email') = 'comunicacao@unifort.com.br');
create policy "Administrador altera arquivos" on storage.objects for update to authenticated using (bucket_id = 'biblioteca-assets' and (auth.jwt() ->> 'email') = 'comunicacao@unifort.com.br');
create policy "Administrador exclui arquivos" on storage.objects for delete to authenticated using (bucket_id = 'biblioteca-assets' and (auth.jwt() ->> 'email') = 'comunicacao@unifort.com.br');
```

## Observação importante de segurança

No início, qualquer pessoa autenticada no seu Supabase poderá administrar a lista. Crie contas somente para administradores. Se futuramente houver vários tipos de usuários, podemos restringir as regras por uma lista de e-mails/perfis.

## Como cadastrar um vídeo

Abra `.../biblioteca-videos/admin.html`, faça login, clique em **Novo vídeo**, preencha os campos e clique em **Salvar vídeo**. Para vídeos do YouTube, basta colar o link. A capa pode ser uma URL de imagem pública; se não preencher, o card continua funcionando com uma capa neutra.
