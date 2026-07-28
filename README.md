# KizeTech - Lê Văn Trí

KizeTech là blog tĩnh song ngữ về **AI Engineering, DevOps, hạ tầng và tự động hóa quy trình**, được xây dựng bằng Astro 5 và triển khai trên Vercel.

README này mô tả trạng thái hiện tại của repository. Hệ thống cũ từng hỗ trợ nhiều ngôn ngữ và nhiều nhóm nội dung; phiên bản hiện tại chỉ xuất bản **English (`en`)** và **Tiếng Việt (`vi`)**.

## Tổng quan kiến trúc

```text
Markdown (src/content/blog/{en,vi})
             │
             ▼
Astro Content Collection + Zod schema
             │
             ├── Pages/layouts/components
             ├── i18n UI (en, vi)
             ├── SEO metadata + JSON-LD + hreflang
             └── ad placement policy
             │
             ▼
Static build (dist/)
             │
             ▼
Vercel
  ├── static pages and assets
  ├── redirects/rewrites
  └── serverless 410 response for retired content
```

Phần lớn website là HTML tĩnh được sinh lúc build. Hai Vercel Functions trong `api/` chỉ phục vụ trang `410 Gone` cho URL, locale hoặc category đã ngừng hoạt động.

## Công nghệ và đặc điểm chính

- Astro 5, TypeScript và Astro Content Collections.
- Nội dung Markdown song ngữ trong `src/content/blog/en` và `src/content/blog/vi`.
- Route tĩnh theo locale, bài viết, category và phân trang.
- SEO tích hợp trong layout: canonical, `hreflang`, Open Graph, Twitter Card và JSON-LD.
- Sitemap tĩnh, robots.txt và các script gửi URL lên Google/Bing.
- Registry chống đăng trùng slug, nguồn và ảnh.
- Ảnh bên ngoài được quản lý chủ yếu qua Cloudinary; Markdown có remark plugin để lazy-load ảnh.
- Quảng cáo được cấu hình tách biệt giữa nội dung, chính sách vị trí và component render.
- Vercel trả HTTP `410 Gone` cho nội dung đã gỡ thay vì để thành soft 404.

## Sơ đồ kiến trúc hệ thống

Bốn sơ đồ kiến trúc mô tả toàn bộ hệ thống từ khâu tiếp nhận bài viết đến phát hành và thông báo công cụ tìm kiếm:

| # | Tài liệu | Mô tả |
|---|----------|--------|
| 1 | **Sơ đồ Use Case** | Tổng quan 18 use case, 5 actor, 4 nhóm chức năng (A–D). Xem chi tiết **tại đây**: https://github.com/levantri10/kizetech_blog/blob/main/use_case_kizetech.md |
| 2 | **Sơ đồ Hoạt động (Activity)** | Luồng chi tiết từng stage: tiếp nhận ảnh → upload Cloudinary → Gate 2/3 → build → deploy → notify. Xem chi tiết **tại đây**: https://github.com/levantri10/kizetech_blog/blob/main/activity_kizetech.md |
| 3 | **Sơ đồ Kiến trúc Kiểm duyệt** | Chi tiết Gate 2 (content-check: frontmatter, 4-section, checklist A/B/C) và Gate 3 (11 rules: 7 biên tập + 4 cơ khí). Xem chi tiết **tại đây**: https://github.com/levantri10/kizetech_blog/blob/main/moderation_architecture_kizetech.md |
| 4 | **Sơ đồ Tech Stack & Kiến trúc** | Toàn bộ tech stack: Authoring (Cursor) → 4-stage Pipeline → Vercel Runtime → JSON-LD Schema → External Services. Xem chi tiết **tại đây**: https://github.com/levantri10/kizetech_blog/blob/main/techstack_architecture_kizetech.md |

---

## Cấu trúc repository

```text
.
├── api/
│   ├── gone.mjs                 # Vercel Function trả trang 410
│   └── retired-locale.mjs       # Alias tới handler 410
├── public/                      # favicon, logo, robots.txt, sitemap.xml
├── scripts/
│   ├── data/
│   │   └── published-registry.json
│   ├── lib/content-registry.mjs
│   ├── generate-sitemap.mjs
│   ├── submit-index.mjs
│   ├── submit-bing.mjs
│   └── ...                      # công cụ nội dung, ảnh và bảo trì
├── src/
│   ├── components/              # UI, quảng cáo, bài liên quan, calculator
│   ├── config/
│   │   ├── site.ts              # domain, publisher, editorial focus
│   │   ├── ad-config.ts         # thông tin ad network
│   │   └── ad-policy.ts         # vị trí được phép hiển thị quảng cáo
│   ├── content/
│   │   ├── config.ts            # schema frontmatter
│   │   └── blog/{en,vi}/        # nội dung đang xuất bản
│   ├── i18n/                    # locale UI và danh sách locale/category nghỉ
│   ├── layouts/                 # base, bài viết và trang Gone
│   ├── pages/                   # Astro file-based routes
│   ├── styles/
│   └── utils/                   # truy vấn bài, slug, path và Markdown plugin
├── astro.config.mjs
├── vercel.json                  # build, cache, redirect và rewrite/410
└── package.json
```

Các thư mục `.astro/`, `dist/` và `node_modules/` là output/cache sinh tự động, không phải source cần chỉnh sửa trực tiếp.

## Route hiện tại

| URL | Chức năng |
| --- | --- |
| `/` | Chuyển về `/en/` bằng HTML redirect |
| `/{lang}/` | Trang chủ và trang đầu của feed |
| `/{lang}/page/{n}` | Phân trang, 5 bài mỗi trang |
| `/{lang}/blog/{slug}` | Chi tiết bài viết |
| `/{lang}/category/{slug}` | Danh sách bài theo category đang có nội dung |
| `/{lang}/author` | Trang tác giả |
| `/{lang}/organization` | Thông tin đơn vị xuất bản |
| `/{lang}/privacy` | Chính sách riêng tư |
| `/{lang}/terms` | Điều khoản |

Trong bảng trên, `{lang}` chỉ nhận `en` hoặc `vi`.

Các route `about` và `contact` được Vercel redirect về `author`; route `categories` được redirect về trang chủ. Locale đã nghỉ và category không còn nội dung được xử lý trong `vercel.json`, thường trả trang 410 rồi chuyển người dùng về `/en/`.

> Lưu ý: hành vi redirect/rewrite/HTTP 410 đầy đủ chỉ có trên Vercel. `astro dev` và `astro preview` không mô phỏng toàn bộ `vercel.json`.

## Cài đặt và chạy local

Yêu cầu Node.js 18.17 trở lên; khuyến nghị dùng Node.js 20 LTS.

```bash
npm ci
npm run dev
```

Mở `http://localhost:4321`. Các lệnh chính:

```bash
npm run dev       # development server
npm run build     # build website tĩnh vào dist/
npm run preview   # xem bản build local
```

`npm ci` nên được dùng để cài đúng dependency từ `package-lock.json`, bao gồm image service `sharp` mà Astro cần khi build.

## Thêm hoặc cập nhật bài viết

Mỗi bài là một file Markdown tại:

```text
src/content/blog/en/<slug>.md
src/content/blog/vi/<slug>.md
```

Frontmatter hợp lệ:

```markdown
---
title: "Tiêu đề bài viết"
author: "Lê Văn Trí"
date: today
image: "https://res.cloudinary.com/.../image.webp"
excerpt: "Mô tả ngắn dùng trên card và metadata."
category: "technology"
subcategory: "DevOps"
featured: false
draft: false
tags: ["AI", "DevOps"]
language: "vi"
translationGroup: "stable-id-shared-by-en-and-vi"
readingTime: 8
noAds: false
showCalculator: false
---

Nội dung Markdown...
```

Các trường bắt buộc là `title`, `author`, `date`, `excerpt` và `category`. `image` là tùy chọn nhưng nếu có phải là URL hợp lệ.

`category` phải thuộc một trong các giá trị schema hiện có:

```text
travel, food, technology, lifestyle, business, health,
education, entertainment, sports, culture, world
```

Tuy vậy định hướng biên tập hiện tại là `technology`; một số category cũ được đánh dấu retired và được Vercel trả 410 khi không còn bài.

Quy tắc khi tạo cặp bài song ngữ:

1. Đặt đúng `language: "en"` hoặc `language: "vi"`.
2. Hai bài tương ứng nên dùng cùng `translationGroup`.
3. Tên file là slug trên URL; có thể khác nhau giữa hai ngôn ngữ.
4. Không sửa trực tiếp `public/sitemap.xml`; sinh lại bằng script sau khi nội dung thay đổi.
5. Chạy build để Astro kiểm tra schema và toàn bộ static route.

```bash
npm run sitemap
npm run build
```

`draft: true` loại bài khỏi feed và route production. `showCalculator: true` bật calculator trong layout bài viết. Trường `noAds` đã có trong schema nhưng layout hiện chưa đọc trường này; việc tắt quảng cáo vẫn do `ad-policy.ts` và loại trang quyết định.

## Luồng nội dung và registry

Repository có pipeline thử nghiệm theo bốn pha:

```text
news-scraper → scripts/data/raw/*.json → publisher → Markdown → Astro build
```

Registry tại `scripts/data/published-registry.json` theo dõi slug, URL nguồn và URL ảnh. Mặc định cùng một nguồn hoặc ảnh chỉ được dùng tối đa hai lần. Sau khi chỉnh nội dung thủ công có thể đồng bộ lại bằng:

```bash
npm run registry:sync
```

### Trạng thái các script cũ

`scripts/news-scraper.mjs`, `scripts/publisher.mjs` và `scripts/translate.mjs` vẫn chứa danh sách 10–11 locale từ kiến trúc cũ. Trong khi đó schema và UI production chỉ chấp nhận `en`/`vi`. Vì vậy:

- không chạy `publish`, `publish:all`, `pipeline:*` hoặc `translate` trực tiếp trên production content nếu chưa sửa target language về `en`/`vi`;
- output cho `zh`, `es`, `ar`, ... sẽ không qua schema build hiện tại;
- `news-scraper.mjs` hiện tạo dữ liệu mẫu cho discovery, chưa phải crawler production hoàn chỉnh.

Các npm script này được giữ để bảo trì/chuyển đổi dần, không phải happy path để xuất bản bài mới.

## Ảnh và Cloudinary

Các công cụ upload/search/fix ảnh sử dụng một hoặc nhiều biến môi trường sau:

```bash
export CLOUDINARY_CLOUD_NAME="..."
export CLOUDINARY_API_KEY="..."
export CLOUDINARY_API_SECRET="..."
# hoặc
export CLOUDINARY_URL="cloudinary://..."
```

Các lệnh bảo trì thường dùng:

```bash
npm run validate:images
npm run fix:images
npm run upload
```

Hãy đọc source của script tương ứng trước khi chạy vì một số script được viết cho migration cụ thể và có thể cập nhật hàng loạt Markdown hoặc tài nguyên Cloudinary.

## Sitemap và indexing

Sinh lại sitemap:

```bash
npm run sitemap
```

Gửi URL lên Google Indexing API:

```bash
npm run index
node scripts/submit-index.mjs --url "https://kizetech.me/en/blog/example"
```

Script Google tự tìm service-account JSON tại `google console/*.json`, sau đó mới thử tên file legacy ở root. Không commit credential.

Gửi URL lên Bing:

```bash
export BING_INDEXING_KEY="..."
npm run index:bing
```

Bing key cũng có thể đặt tại `google console/bing-api-key.txt`. Cả hai script ưu tiên lấy danh sách URL từ `public/sitemap.xml`.

## Biến môi trường

Website tĩnh không cần secret để chạy local. Secret chỉ cần cho tooling:

| Biến | Dùng bởi |
| --- | --- |
| `CLOUDINARY_CLOUD_NAME` | upload và quản lý ảnh |
| `CLOUDINARY_API_KEY` | upload và quản lý ảnh |
| `CLOUDINARY_API_SECRET` | upload và quản lý ảnh |
| `CLOUDINARY_URL` | cấu hình Cloudinary dạng URL |
| `BING_INDEXING_KEY` | Bing URL Submission API |
| `TRANSLATE_API_KEY` | script dịch legacy |
| `TRANSLATE_PROVIDER` | provider dịch; thực tế hiện chỉ OpenAI được implement |
| `FORCE_REWRITE=true` | cho publisher legacy ghi đè bài |

Không đưa API key, service-account JSON hoặc file credential vào Git.

## Quảng cáo

Quảng cáo được chia thành ba lớp:

- `src/config/ad-config.ts`: network URL và kích thước format.
- `src/config/ad-policy.ts`: bật/tắt vị trí theo loại trang.
- `src/components/AdSlot.astro` và các component liên quan: lazy loading/render.

Trang trust như author, organization, privacy và terms không hiển thị quảng cáo. Khi thay network hoặc vị trí, cập nhật config/policy thay vì hard-code thêm script vào page.

## Deploy Vercel

Vercel dùng cấu hình:

```text
build command: npm run build
output:        dist
framework:     astro
install:       npm install
```

`vercel.json` còn chịu trách nhiệm:

- cache asset fingerprint trong `/_astro/*`;
- redirect các URL hợp nhất;
- rewrite nội dung/locale đã nghỉ sang API trả 410;
- phục vụ robots.txt cho route theo locale.

Sau thay đổi route hoặc chính sách gỡ nội dung, cần kiểm tra cả Astro static paths lẫn thứ tự rule trong `vercel.json`. Không tạo static page cho URL cần trả 410.

## Checklist trước khi merge/deploy

```bash
npm ci
npm run registry:sync
npm run sitemap
npm run build
```

Sau deploy, kiểm tra tối thiểu:

- `/`, `/en/` và `/vi/`;
- một bài song ngữ và language switcher;
- canonical/hreflang của bài;
- một category đang hoạt động;
- một URL retired trả HTTP 410 trên Vercel;
- `/robots.txt` và `/sitemap.xml`.
