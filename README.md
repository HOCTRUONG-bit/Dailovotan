# Đạo Lộ Vô Tận — Web/PWA v2

Giao diện cổ phong được tối ưu cho màn hình điện thoại. Có manifest và service worker để hỗ trợ cài lên màn hình chính và cache phần lõi sau lần truy cập đầu. Game hiện có tu luyện, đột phá, công pháp, luyện đan, bí cảnh, túi đồ, save cục bộ và tùy chọn tích hợp tài khoản cloud bằng Supabase.

## Chạy thử
Do PWA/service worker và module JavaScript cần origin an toàn, hãy chạy qua HTTPS hosting hoặc localhost; mở `index.html` bằng `file://` sẽ không kích hoạt đầy đủ PWA.

## Đưa game lên web miễn phí
1. Tạo repository GitHub và tải toàn bộ file/thư mục này lên.
2. Vào Settings → Pages → Build and deployment, chọn deploy từ branch chính và thư mục root.
3. Mở URL HTTPS được cấp trên Android Chrome.
4. Chrome menu ⋮ → **Thêm vào màn hình chính** hoặc **Cài đặt ứng dụng** (tên mục tùy phiên bản).

## Bật tài khoản và đồng bộ nhiều thiết bị (Supabase)
Phần đăng nhập đã có giao diện và mã tích hợp, nhưng cần cấu hình dự án riêng trước khi hoạt động.

1. Tạo dự án tại https://supabase.com.
2. Trong SQL Editor, chạy:

```sql
create table public.game_saves (
  user_id uuid primary key references auth.users(id) on delete cascade,
  save_data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.game_saves enable row level security;

create policy "Users can read their own save"
on public.game_saves for select
to authenticated
using (auth.uid() = user_id);

create policy "Users can insert their own save"
on public.game_saves for insert
to authenticated
with check (auth.uid() = user_id);

create policy "Users can update their own save"
on public.game_saves for update
to authenticated
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

3. Mở `config.js` và điền Project URL cùng **anon/public key** lấy từ phần Project Settings → API của Supabase:

```js
window.APP_CONFIG = {
  SUPABASE_URL: "https://YOUR_PROJECT.supabase.co",
  SUPABASE_ANON_KEY: "YOUR_SUPABASE_ANON_PUBLIC_KEY"
};
```

4. Trong Supabase Authentication, cấu hình Site URL và Redirect URLs đúng URL website HTTPS. Nếu xác nhận email được bật, người chơi cần xác nhận email trước khi đăng nhập.
5. Tải lại website. Tạo tài khoản/đăng nhập; game sẽ tải save từ cloud hoặc tạo save cloud lần đầu.

**Bảo mật:** chỉ dùng anon/public key ở frontend và bật RLS như hướng dẫn. Tuyệt đối không đưa `service_role` key vào `config.js`. Không có máy chủ riêng nào được triển khai tự động bởi ZIP này.

## Lưu ý
- Lưu cục bộ dùng localStorage trên từng trình duyệt; xóa dữ liệu trình duyệt có thể mất save cục bộ.
- Đồng bộ nhiều thiết bị chỉ hoạt động sau khi bạn tự cấu hình Supabase và tạo bảng/policy.
- Service worker cache giao diện, không đảm bảo toàn bộ chức năng hoạt động offline vì thư viện Supabase được tải từ CDN.
- Đây vẫn là prototype; cần kiểm thử trên hosting thực tế trước khi phát hành rộng rãi.
