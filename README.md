# PERSONAL ARCHIVE — 설치 안내

`index.html` 한 파일이 사이트 전체입니다. GitHub Pages에 올리고, 로그인·저장은 Supabase(무료)에 연결합니다.

---

## 1. Supabase 준비 (10분, 한 번만)

1. [supabase.com](https://supabase.com) 가입 → **New project** 생성 (region은 Northeast Asia 권장).
2. 왼쪽 메뉴 **SQL Editor** → 아래 내용을 붙여넣고 **Run**.

```sql
-- 기록 테이블
create table if not exists archive_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  payload jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);
alter table archive_data enable row level security;

create policy "own row read"   on archive_data for select using (auth.uid() = user_id);
create policy "own row insert" on archive_data for insert with check (auth.uid() = user_id);
create policy "own row update" on archive_data for update using (auth.uid() = user_id) with check (auth.uid() = user_id);

-- 이미지 버킷
insert into storage.buckets (id, name, public) values ('gallery','gallery',true)
on conflict (id) do nothing;

create policy "gallery read"   on storage.objects for select using (bucket_id = 'gallery');
create policy "gallery upload" on storage.objects for insert to authenticated
  with check (bucket_id = 'gallery' and (storage.foldername(name))[1] = auth.uid()::text);
create policy "gallery update" on storage.objects for update to authenticated
  using (bucket_id = 'gallery' and (storage.foldername(name))[1] = auth.uid()::text);
create policy "gallery delete" on storage.objects for delete to authenticated
  using (bucket_id = 'gallery' and (storage.foldername(name))[1] = auth.uid()::text);
```

3. **Authentication → Sign In / Providers → Email**
   - *Confirm email* 를 **끄면** 가입 즉시 로그인됩니다. 켜두면 메일 인증 후 로그인.
   - 본인 계정을 만든 뒤에는 **Allow new users to sign up** 을 꺼두세요. 남이 가입할 수 없게 됩니다.
4. **Project Settings → API** 에서 두 값을 복사: **Project URL**, **anon public key**.

> anon key는 공개돼도 되는 값입니다. 위에서 켠 RLS 정책이 있어 다른 사람은 내 데이터를 읽지 못합니다.

---

## 2. GitHub Pages 올리기

1. 새 저장소 생성 → 아래 파일을 모두 같은 위치(저장소 최상단)에 업로드합니다.
   - `index.html` — 사이트 전체
   - `manifest.webmanifest` — 홈 화면 앱 설정
   - `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `favicon-32.png` — 아이콘
   - `README.md` (선택)
2. 저장소 **Settings → Pages** → Source: `Deploy from a branch`, Branch: `main` / `root` → Save.
3. 1~2분 뒤 `https://아이디.github.io/저장소이름/` 주소가 열립니다.

### 홈 화면에 추가하기
- **아이폰** — 사파리로 사이트를 열고 공유 버튼 → *홈 화면에 추가*. 검은 바탕에 흰 스파클과 파란 점이 있는 아이콘으로 추가되고, 브라우저 주소창 없이 앱처럼 열립니다.
- **안드로이드** — 크롬 메뉴 → *앱 설치* 또는 *홈 화면에 추가*.
- 아이콘을 바꾸면 이미 추가해 둔 것은 그대로 남습니다. 홈 화면에서 지웠다가 다시 추가해 주세요.

### 아이디/비번 화면을 바로 띄우려면
`index.html` 위쪽 `CONFIG` 부분을 수정해 커밋하세요. 그러면 어느 기기에서 열어도 곧장 로그인 화면이 나옵니다.

```js
const CONFIG = {
  SUPABASE_URL: "https://xxxx.supabase.co",
  SUPABASE_ANON_KEY: "eyJhbGciOi...",
  BUCKET: "gallery",
};
```

비워둔 채 올려도 됩니다. 첫 화면에서 두 값을 입력하면 그 브라우저에 기억됩니다.

---

## 3. 사용

- **계정 만들기** 탭에서 이메일·비밀번호로 가입 → 이후 어느 기기에서든 같은 아이디로 로그인하면 기록이 이어집니다.
- **01 D-DAY** — 시작일이 1일째. 다음 100일 단위·N주년까지 남은 날이 함께 표시됩니다.
- **02 GALLERY** — 파일 업로드(자동 압축 후 Supabase Storage 저장) 또는 이미지 주소 등록. 페어·태그로 필터.
- **03 LOGS** — 글 작성·검색, 페어와 태그 연결.
- **04 SETTINGS** — 포인트 컬러(기본 블루), 아카이브 이름, JSON 백업 내려받기/불러오기, 로그아웃.

연결 없이 **이 브라우저에만 저장** 모드로도 쓸 수 있지만, 그 브라우저를 벗어나면 기록이 보이지 않습니다.

---

## 참고

- 이미지 버킷을 `public`으로 두었기 때문에 이미지 URL을 아는 사람은 파일을 볼 수 있습니다(목록은 노출되지 않음). 완전 비공개가 필요하면 버킷을 private으로 바꾸고 signed URL 방식으로 바꿔야 합니다.
- 무료 플랜 기준 저장소 1GB, 데이터베이스 500MB. 개인 아카이브에는 충분합니다.
