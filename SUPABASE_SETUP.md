# Kích hoạt tài khoản đồng bộ (Supabase, miễn phí)

Web đã có sẵn tính năng **đăng nhập bằng email (không mật khẩu)** và **tự đồng bộ tiến độ** giữa các thiết bị.
Để bật, cần một dự án Supabase (gói miễn phí đủ dùng). Làm một lần, khoảng 10 phút.

## 1. Tạo dự án
1. Vào https://supabase.com → **Start your project** → đăng nhập bằng GitHub.
2. **New project**: đặt tên `hoc-tieng-trung`, chọn khu vực **Southeast Asia (Singapore)**, đặt mật khẩu cơ sở dữ liệu (cất giữ riêng, không gửi cho ai).

## 2. Tạo bảng lưu tiến độ
Vào **SQL Editor** → **New query** → dán đoạn dưới → **Run**:

```sql
create table public.progress (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null,
  updated_at timestamptz not null default now()
);
alter table public.progress enable row level security;

create policy "Đọc tiến độ của chính mình" on public.progress
  for select using (auth.uid() = user_id);
create policy "Tạo tiến độ của chính mình" on public.progress
  for insert with check (auth.uid() = user_id);
create policy "Cập nhật tiến độ của chính mình" on public.progress
  for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

Các dòng `policy` (RLS) đảm bảo **mỗi người chỉ đọc/ghi được tiến độ của chính mình**.

## 3. Cấu hình đăng nhập
**Authentication → URL Configuration**:
- **Site URL**: `https://eng072291.github.io/hoc-tieng-trung/`
- **Redirect URLs** → Add URL: `https://eng072291.github.io/hoc-tieng-trung/**`
  (thêm cả tên miền riêng nếu sau này dùng, ví dụ `https://tiengtrung.pages.dev/**`)

**Authentication → Sign In / Providers → Email**: bật **Enable Email provider** (mặc định đã bật).

## 4. Lấy 2 thông tin gửi cho người lập trình
**Project Settings → API** (hoặc **Data API**):
- **Project URL**, dạng `https://xxxxxxxx.supabase.co`
- **anon public key** (chuỗi dài bắt đầu bằng `eyJ...` hoặc `sb_publishable_...`)

Khóa *anon/publishable* được thiết kế để nằm công khai trong web và an toàn khi đã bật RLS ở bước 2.
**Tuyệt đối không gửi** khóa `service_role` / `secret` hay mật khẩu cơ sở dữ liệu.

Điền 2 giá trị này vào `index.html`:
```js
const SUPA_URL='https://xxxxxxxx.supabase.co';
const SUPA_KEY='eyJ...';
```

## Ghi chú
- Gói miễn phí giới hạn số email đăng nhập gửi mỗi giờ. Khi có nhiều người dùng, nên cấu hình SMTP riêng trong **Authentication → Emails → SMTP Settings**.
- Dự án miễn phí sẽ tạm dừng nếu không có ai dùng trong 7 ngày; mở lại bằng nút **Restore** trên trang Supabase.
