# 营销排期表 · Supabase 配置指南

按下面步骤在 Supabase 创建项目并拿到密钥，再在页面中填入即可启用「账号登录 + 云端数据库」。

---

## 第一步：注册并创建项目

1. 打开 **https://supabase.com**，用 GitHub 或邮箱注册并登录。
2. 点击 **New Project**。
3. **Name** 填：`marketing-scheduler`（或任意名称）。
4. **Database Password** 设一个强密码并**记住**（以后连数据库会用到）。
5. **Region** 选离你近的（如 Singapore）。
6. 点击 **Create new project**，等待约 1 分钟创建完成。

---

## 第二步：创建数据表

1. 左侧点 **SQL Editor**。
2. 点击 **New query**，把下面整段 SQL 粘贴进去，然后点 **Run**（或 Ctrl+Enter）。

```sql
-- 排期表（每个用户只看自己的数据）
create table if not exists public.schedules (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  title text not null,
  platforms jsonb default '[]',
  start_date date not null,
  end_date date not null,
  scheduled_time text default '12:00',
  reminder_minutes int default 30,
  status text default 'pending',
  assignee text default '',
  description text default '',
  reminded boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- 行级安全：用户只能操作自己的排期
alter table public.schedules enable row level security;

create policy "用户只能管理自己的排期"
  on public.schedules for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

-- 操作记录表（可选，用于记录创建/修改/删除）
create table if not exists public.activity_logs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references auth.users(id) on delete cascade,
  action text not null,
  schedule_id uuid references public.schedules(id) on delete set null,
  details jsonb default '{}',
  created_at timestamptz default now()
);

alter table public.activity_logs enable row level security;

create policy "用户只能看自己的操作记录"
  on public.activity_logs for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

3. 若没有报错，说明表已创建成功。

---

## 第三步：关闭邮箱确认（可选，方便本地测试）

1. 左侧点 **Authentication** → **Providers**。
2. 找到 **Email**，点开。
3. 把 **Confirm email** 关掉（以后要邮箱验证再打开）。
4. 点 **Save**。

---

## 第四步：拿到 API 密钥

1. 左侧点 **Project Settings**（齿轮图标）。
2. 点 **API**。
3. 在 **Project URL** 处点击复制，得到类似：  
   `https://xxxxxxxxxxxx.supabase.co`
4. 在 **Project API keys** 里找到 **anon public**，点击复制（一长串字母数字）。

---

## 第五步：在项目里填入密钥

1. 用编辑器打开项目里的 **index.html**。
2. 搜索：`YOUR_SUPABASE_URL`
3. 把 `YOUR_SUPABASE_URL` 替换成你复制的 **Project URL**（保留引号）。
4. 搜索：`YOUR_SUPABASE_ANON_KEY`
5. 把 `YOUR_SUPABASE_ANON_KEY` 替换成你复制的 **anon public** 密钥（保留引号）。
6. 保存文件。

例如替换后类似：

```javascript
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## 完成

刷新排期表页面。若配置正确，会先看到**登录/注册**界面；注册或登录后会进入排期表，数据会保存在 Supabase，并随账号走。

- **未配置或密钥仍为占位符**：页面会继续使用本地 localStorage，不显示登录。
- **已配置且已登录**：排期数据在云端，换设备登录同一账号即可看到同一份数据。

如有报错，可看浏览器控制台（F12 → Console）里的错误信息，或检查 Supabase 的 **Authentication** → **Users** 和 **Table Editor** → **schedules** 是否有数据。
