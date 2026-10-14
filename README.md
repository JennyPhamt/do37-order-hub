# Đô 37 B2B Order Hub

Prototype có thể thao tác cho solution của **HTX Nông Sản Sạch Đô 37** — Symptom 3: thông tin đơn B2B nằm rải rác ở
Call, Zalo, Messenger nên ghi nhận chậm và thiếu người phụ trách.

> **Đơn vào một nơi. Người xử lý rõ ràng.** Đại lý đặt hàng, Đô 37 ghi nhận và theo dõi từng bước trong cùng một hệ
> thống.

Luồng đầy đủ: đại lý đặt đơn → đơn xuất hiện ở dashboard → nhân viên xác nhận (nhận phụ trách) → chuẩn bị → gửi hàng →
ghi nhận đại lý đã nhận hàng. Nhân viên nhập được đơn từ kênh khác vào cùng danh sách.

**Đây là bản demo cho capstone.** Dữ liệu là dữ liệu mẫu, lưu trên trình duyệt; không gửi email thật; số liệu báo cáo
tính từ dữ liệu demo, không phải kết quả validation hay pilot.

---

## 1. Chạy dự án

Yêu cầu: Node.js 20.9 trở lên (đã kiểm tra với Node 22), npm.

```bash
npm install
npm run dev          # http://localhost:3000
```

Bản production:

```bash
npm run build
npm start            # http://localhost:3000 (đổi cổng: PORT=3100 npm start)
```

Không cần biến môi trường để chạy demo. Nếu muốn, sao chép `.env.example` thành `.env.local`
(`NEXT_PUBLIC_DATA_MODE=demo`).

Kiểm tra chất lượng:

```bash
npm test             # 29 unit test (Vitest): quy tắc đơn, KPI, giờ VN, repository demo
npm run lint         # ESLint (cấu hình Next.js)
npm run typecheck    # TypeScript
npm run format       # Prettier
```

## 2. Kịch bản trình bày gợi ý (khoảng 5 phút)

Bấm **“Khôi phục dữ liệu demo”** trên thanh vàng trước khi trình bày — các mốc thời gian mẫu được tính lại theo giờ hiện
tại, nên “đơn mới”, “quá hạn” luôn đúng ngữ cảnh. Mở hai tab cùng trình duyệt: một tab đại lý, một tab nội bộ.

| Bước | Màn hình | Thao tác |
| --- | --- | --- |
| 1 | `/` | Giới thiệu vấn đề và cách hoạt động. |
| 2 | `/dealer/access` | Bấm “Điền mã mẫu” của **Đại lý Mẫu A** → Tiếp tục. (Thử mã của **Đại lý Mẫu D** để thấy thông báo ngừng hoạt động.) |
| 3 | `/dealer/new` | Thêm 3 sản phẩm, thử chọn trùng sản phẩm để thấy nút “Gộp số lượng”, chọn ngày giao → Xem lại → Gửi. |
| 4 | Xác nhận | Trang “Đơn hàng đã được ghi nhận”, có mã đơn để sao chép và lưu ý “đã ghi nhận ≠ đã xác nhận ngày giao”. |
| 5 | `/admin/login` (tab 2) | “Trải nghiệm nhân viên” → đơn vừa đặt có ngay trên dashboard. Thẻ “Quá hạn” lọc đơn trễ hẹn. |
| 6 | Chi tiết đơn | Xác nhận đơn (hẹn giờ giao, tự nhận phụ trách) → Bắt đầu chuẩn bị → Đã gửi → Đã giao. Xem timeline “Ai làm gì, lúc nào”. Tab đại lý tự cập nhật trạng thái. |
| 7 | `/admin/orders/new` | Nhập đơn hộ kênh Zalo, thời gian nhận 2 giờ trước → hệ thống bắt ghi lý do nhập trễ, lưu riêng hai mốc thời gian. |
| 8 | `/admin/reports` | 7 chỉ số, có mẫu số và số đơn; chọn kỳ không có dữ liệu để thấy “Chưa đủ dữ liệu”. Xuất CSV. |

Công cụ demo:

- Ở bước xem lại đơn: “Công cụ demo: mô phỏng sự cố khi gửi” → **Mất phản hồi sau khi lưu** → bấm Gửi (báo lỗi) → bấm
  Gửi lại: vẫn chỉ có **một** đơn (idempotency key).
- Cài đặt → “Mô phỏng lỗi gửi email”: đơn mới vẫn được lưu, outbox ghi trạng thái lỗi và cho thử lại.
- “Đổi vai trò demo” ở sidebar để chuyển nhân viên ↔ quản lý. Quản lý có thêm: quản lý đại lý/sản phẩm/nhân viên/cài
  đặt và “Điều chỉnh” (lùi trạng thái, sửa mốc thời gian, đổi người phụ trách — luôn kèm lý do, ghi thành sự kiện mới).
- Quản lý → Đại lý → “Đổi mã” Đại lý Mẫu A: tab đại lý bị đưa về trang nhập mã với thông báo mã đã thu hồi.

## 3. Đang demo và đã nối backend

| Phần | Bản demo hiện tại | Đã nối backend? | Khi triển khai pilot |
| --- | --- | --- | --- |
| Đơn hàng, đại lý, sản phẩm, cài đặt | localStorage của trình duyệt, đọc sau khi hydrate | **Chưa** | Supabase Postgres — bản nháp `supabase/schema.sql` |
| Mã đại lý | Kiểm tra trong trình duyệt (SHA-256), khóa 60 giây sau 5 lần sai | **Chưa** | Route Handler phía server: HMAC + secret, giới hạn lần thử, cookie httpOnly gắn với `dealer_access` |
| Phiên đại lý | Lưu localStorage; đổi mã/ngừng đại lý thì phiên hết hiệu lực | **Chưa** | Bảng `dealer_sessions`, thu hồi khi đổi mã |
| Nhân viên | Nút chọn vai trò — **không phải bảo mật thật** | **Chưa** | Supabase Auth (email + mật khẩu) + bảng `profiles` + RLS |
| Tạo đơn, chuyển trạng thái | Repository mô phỏng các kiểm tra phía server | **Chưa** | RPC trong transaction (`create_web_order`, `create_staff_order`, `confirm_order`, `mark_delivered`; các bước còn lại theo cùng khuôn) |
| Email báo đơn mới | Tạo bản xem trước trong outbox, **không gửi** | **Chưa** | Worker đọc `notification_outbox`, gửi sau khi commit, có retry |
| Báo cáo, CSV | Tính từ dữ liệu demo | **Chưa** | Tính từ đơn thực tế |

Không chức năng nào đang nối backend. `src/repositories/supabase/index.ts` hiện chỉ báo lỗi “chưa cài đặt” nếu đặt
`NEXT_PUBLIC_DATA_MODE=supabase`.

## 4. Cấu trúc thư mục

```
src/
  app/                       Các trang (Next.js App Router)
    page.tsx                 / — giới thiệu solution
    dealer/access/           /dealer/access — nhập mã đại lý
    dealer/(portal)/         khung đại lý (kiểm tra phiên) + new, history, orders/[id]
    admin/login/             /admin/login — đăng nhập demo
    admin/(panel)/           khung nội bộ (sidebar, kiểm tra vai trò) + dashboard, orders/new,
                             orders/[id], dealers, products, reports, settings
  components/
    ui/                      Button, Field, Card, Badge, Dialog, Toast, StatTile…
    order/                   form dòng sản phẩm, tiến trình trạng thái, timeline, badge
    layout/                  Logo, thanh “Bản demo”
  domain/                    Quy tắc nghiệp vụ, không phụ thuộc giao diện
    types.ts                 Kiểu dữ liệu (theo data model)
    status.ts                Trạng thái, bước kế tiếp hợp lệ (không nhảy cóc)
    order-rules.ts           Kiểm tra đơn, gộp dòng trùng, tổng theo đơn vị
    time.ts                  Giờ Việt Nam (UTC+7) ↔ UTC
    schedule.ts              Giờ làm việc, cut-off → lịch xử lý dự kiến
    alerts.ts                Quá hạn giao, chờ xác nhận quá hạn, mở quá 24 giờ
    kpi.ts                   7 chỉ số báo cáo
    csv.ts                   Xuất CSV UTF-8 BOM
  repositories/
    types.ts                 Interface dùng chung cho demo và Supabase
    local/                   Bản demo (localStorage) — mô phỏng kiểm tra phía server
    supabase/                Chỗ cài bản Supabase (chưa làm)
  demo/                      Dữ liệu mẫu, mô phỏng sự cố
  hooks/                     useRepoQuery, useNow, useFormSession…
supabase/
  schema.sql                 Bản nháp schema + RLS + RPC cho pilot
  test/                      Stub Supabase + kiểm thử SQL
```

## 5. Quy tắc nghiệp vụ quan trọng

- **Thời gian:** lưu UTC, hiển thị Asia/Ho_Chi_Minh. Mỗi đơn có `received_at` (Đô 37 nhận đơn thực tế) và
  `recorded_at` (lưu vào hệ thống, giờ server). Đơn web hai mốc trùng nhau; đơn nhập hộ nhập `received_at`, quá ngưỡng
  (mặc định 30 phút) phải ghi lý do; không cho thời điểm nhận ở tương lai.
- **Cut-off chỉ quyết định lịch xử lý.** Đơn nhận 21:40 được xếp xử lý từ 07:30 ngày làm việc kế tiếp, nhưng
  `received_at` vẫn là 21:40 và mọi KPI tính từ 21:40. Hệ thống không tự hứa ngày giao — chỉ có ngày giao khi nhân viên
  xác nhận.
- **Trạng thái:** Mới → Đã xác nhận → Đang chuẩn bị → Đã gửi → Đã giao, không nhảy cóc. Xác nhận = nhận phụ trách +
  `confirmed_at` + giờ giao đã hẹn. Đã giao nhập thời điểm đại lý thực sự nhận hàng (có thể sớm hơn lúc cập nhật, không
  trước lúc nhận đơn). Đại lý tự hủy khi đơn còn Mới; nhân viên hủy phải có lý do. Không xóa lịch sử.
- **Số lượng:** số nguyên > 0; sản phẩm trùng phải gộp hoặc đổi; tổng hiển thị theo từng đơn vị (“8 thùng · 10 hộp”),
  không cộng hộp với thùng. Dòng đơn giữ tên/SKU/đơn vị tại thời điểm đặt.
- **Báo cáo:** lọc kỳ theo ngày nhận đơn; đơn hủy không vào mẫu số; đơn thiếu mốc thời gian bị loại khỏi phép tính
  (không coi là 0); mẫu số = 0 → “Chưa đủ dữ liệu”; mẫu < 5 có nhãn “Mẫu nhỏ”. Không có baseline nào được nhập sẵn.
- **Cài đặt** (giờ làm việc, ngày nghỉ, cut-off, ngưỡng, người nhận email) là cấu hình đề xuất, chưa phải chính sách Đô 37.

## 6. Đã kiểm thử những gì

| Tiêu chí nghiệm thu | Bằng chứng |
| --- | --- |
| Đơn 3 sản phẩm lưu đủ 3 dòng | Unit test, E2E, kiểm thử SQL |
| Gửi lại cùng yêu cầu không tạo 2 đơn | Unit test (cả trường hợp mất phản hồi sau khi lưu), E2E, kiểm thử SQL |
| Nhập đơn cũ giữ đúng hai mốc thời gian | Unit test, E2E, kiểm thử SQL |
| Chuyển trạng thái cập nhật timeline và báo cáo | Unit test, E2E |
| Đơn ngoài giờ giữ nguyên thời gian nhận | Unit test (lead time tính từ 21:40) |
| Không cộng số lượng khác đơn vị | Unit test, E2E |
| Hủy đơn cập nhật danh sách và mẫu số | Unit test |
| Không có dữ liệu thì không hiện 100% | Unit test, E2E |
| Dùng tốt ở 375px | E2E: 12 trang không tràn ngang, nút chính cao 44px |
| Đại lý A không xem/sửa đơn của B | Bản demo: kiểm tra ở repository (unit test + mở URL trực tiếp trong E2E). Bản thực tế: schema chặn `anon` và tài khoản không phải nhân viên (kiểm thử SQL); Route Handler phía server cho đại lý **chưa xây** |
| Email lỗi vẫn giữ đơn | Unit test (outbox trạng thái lỗi, đơn vẫn lưu) |

- **E2E** (Playwright, Chromium, chạy trên bản build production): 53/53 kiểm tra đạt, gồm cả luồng đại lý, nhân viên,
  quản lý đổi mã và xuất CSV có BOM. Script E2E dùng khi phát triển, không nằm trong repo.
- **SQL:** `supabase/schema.sql` đã chạy trên PostgreSQL 16 thuần với stub `auth` (`supabase/test/`), 7 nhóm kiểm thử
  đạt. **Chưa** chạy trên dự án Supabase thật. Cách chạy lại:

  ```bash
  createdb do37_schema_test
  cat supabase/test/00-supabase-stub.sql supabase/schema.sql supabase/test/10-smoke-test.sql \
    | psql -X -q -v ON_ERROR_STOP=1 -d do37_schema_test
  ```

## 7. Giới hạn đã biết

- Dữ liệu demo nằm trong trình duyệt đang dùng: hai máy khác nhau không thấy dữ liệu của nhau. Trình bày đại lý và nội
  bộ trên hai tab cùng một trình duyệt.
- Kiểm tra mã đại lý, phiên và vai trò ở bản demo chạy trên trình duyệt nên **không phải cơ chế bảo vệ thật**.
- Định dạng ô ngày/giờ (dd/mm hay mm/dd, 24 giờ hay AM/PM) theo ngôn ngữ của trình duyệt.
- Dữ liệu mẫu tính theo giờ khôi phục; nếu khôi phục lúc nửa đêm, giờ nhận của vài đơn mẫu sẽ rơi vào ban đêm.
- Next.js 16 (Cache Components) giữ tạm trang vừa rời để quay lại nhanh; form đặt đơn đã gửi được làm mới tự động.

## 8. Đưa lên web để trình bày

Có thể deploy lên Vercel như một dự án Next.js thông thường (không cần biến môi trường cho bản demo). Mỗi người xem có
dữ liệu demo riêng trong trình duyệt của họ.
