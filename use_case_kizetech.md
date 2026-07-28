# SƠ ĐỒ USE CASE — HỆ THỐNG XUẤT BẢN BLOG KIZETECH

> **Phạm vi:** tiếp nhận bài viết → kiểm duyệt bắt buộc → build/deploy → phục vụ EN/VI → thông báo công cụ tìm kiếm

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#EFF6FF','primaryTextColor':'#1E3A8A','primaryBorderColor':'#93C5FD','lineColor':'#475569','secondaryColor':'#F8FAFC','tertiaryColor':'#FFFFFF'}}}%%
flowchart TB
    subgraph SYSTEM["KIZETECH BLOG PUBLISHING & DELIVERY SYSTEM"]

        subgraph GROUP_A["A. Tiếp nhận & chuẩn hóa"]
            UC01["UC-01<br/>Gửi bản thảo và yêu cầu xuất bản"]
            UC02["UC-02<br/>Chuẩn bị ảnh hero trên Cloudinary"]
            UC03["UC-03<br/>Sinh / chuẩn hóa frontmatter và Markdown"]
            UC04["UC-04<br/>Sửa bản thảo bị từ chối"]
        end

        subgraph GROUP_B["B. Hai cổng kiểm duyệt bắt buộc"]
            UC05["UC-05 — Gate 2<br/>Kiểm tra contract nội dung, cấu trúc, i18n/SEO"]
            UC06["UC-06 — Gate 3<br/>Audit 11 tiêu chí Zero-Slop và an toàn"]
            UC07["UC-07<br/>Trả báo cáo vi phạm theo field / rule / dòng"]
        end

        subgraph GROUP_C["C. Phát hành & lập chỉ mục"]
            UC08["UC-08<br/>Phát hành bài đã được duyệt"]
            UC09["UC-09<br/>Tạo sitemap (lastmod + hreflang)"]
            UC10["UC-10<br/>Build Astro và kiểm tra dist/sitemap.xml"]
            UC11["UC-11<br/>Commit, push và triển khai Vercel"]
            UC12["UC-12<br/>Xác minh production và thông báo URL đã đổi"]
        end

        subgraph GROUP_D["D. Trải nghiệm website công khai"]
            UC13["UC-13<br/>Duyệt trang chủ, category, author"]
            UC14["UC-14<br/>Đọc bài viết tĩnh theo /{lang}/blog/{slug}"]
            UC15["UC-15<br/>Chuyển EN ↔ VI theo translationGroup"]
            UC16["UC-16<br/>Trả 410 + noindex cho URL đã nghỉ / không hợp lệ"]
            UC17["UC-17<br/>Cho crawler đọc sitemap, HTML và JSON-LD"]
            UC18["UC-18<br/>Xuất canonical, hreflang, Open Graph và schema.org"]
            NOTE_D["⚠️ Kiểm duyệt và deploy là build-time;<br/>độc giả/crawler chỉ truy cập output đã phát hành trên Vercel."]
        end
    end

    %% External Actors
    ACT_AUTHOR["👤 Tác giả / Biên tập viên"]
    ACT_CLOUDINARY["☁️ Cloudinary"]
    ACT_READER["👤 Độc giả EN / VI"]
    ACT_GITHUB["🔧 GitHub + Vercel"]
    ACT_SEARCH["🔍 Google + IndexNow engines"]

    %% Actor → UC relationships
    ACT_AUTHOR --> UC01
    ACT_AUTHOR --> UC02
    ACT_AUTHOR --> UC04
    ACT_CLOUDINARY -->|"upload / delivery URL"| UC02
    ACT_READER --> UC13
    ACT_READER --> UC14
    ACT_READER --> UC16
    ACT_GITHUB -->|"repository + build platform"| UC11
    ACT_SEARCH -->|"URL_UPDATED / IndexNow"| UC12
    ACT_SEARCH -->|"crawl"| UC17

    %% Include / Extend relationships (dashed)
    UC01 -.->|"<<include>>"| UC02
    UC01 -.->|"<<include>>"| UC03
    UC03 -.->|"<<include>>"| UC05
    UC05 -.->|"<<include>>"| UC06
    UC07 -.->|"<<extend>> [bất kỳ check fail]"| UC05
    UC07 -.->|"<<extend>> [bất kỳ rule fail]"| UC06
    UC07 -.->|"yêu cầu sửa"| UC04
    UC06 -.->|"[PASS]"| UC08
    UC08 -.->|"<<include>>"| UC09
    UC08 -.->|"<<include>>"| UC10
    UC08 -.->|"<<include>>"| UC11
    UC08 -.->|"<<include>>"| UC12
    UC14 -.->|"<<include>>"| UC15
    UC17 -.->|"<<include>>"| UC18
    UC16 -.->|"<<extend>> [URL retired / missing]"| UC14
```

## Ký hiệu

| Ký hiệu | Ý nghĩa |
|----------|----------|
| Đường liền | Actor tham gia use case |
| Nét đứt | Include / Extend relationship |
| Vùng màu | Nhóm chức năng, không phải thứ tự thực thi |

## 18 Use Cases

### A. Tiếp nhận & chuẩn hóa
- **UC-01:** Gửi bản thảo và yêu cầu xuất bản *(bao gồm UC-02, UC-03)*
- **UC-02:** Chuẩn bị ảnh hero trên Cloudinary
- **UC-03:** Sinh / chuẩn hóa frontmatter và Markdown *(dẫn đến Gate 2)*
- **UC-04:** Sửa bản thảo bị từ chối

### B. Hai cổng kiểm duyệt bắt buộc
- **UC-05 (Gate 2):** Kiểm tra contract nội dung, cấu trúc, i18n/SEO
- **UC-06 (Gate 3):** Audit 11 tiêu chí Zero-Slop và an toàn
- **UC-07:** Trả báo cáo vi phạm theo field / rule / dòng *(extend từ Gate 2 & 3)*

### C. Phát hành & lập chỉ mục
- **UC-08:** Phát hành bài đã được duyệt *(bao gồm UC-09 → UC-12)*
- **UC-09:** Tạo sitemap (lastmod + hreflang)
- **UC-10:** Build Astro và kiểm tra dist/sitemap.xml
- **UC-11:** Commit, push và triển khai Vercel
- **UC-12:** Xác minh production và thông báo URL đã đổi

### D. Trải nghiệm website công khai
- **UC-13:** Duyệt trang chủ, category, author
- **UC-14:** Đọc bài viết tĩnh theo /{lang}/blog/{slug} *(có thể extend UC-16)*
- **UC-15:** Chuyển EN ↔ VI theo translationGroup
- **UC-16:** Trả 410 + noindex cho URL đã nghỉ / không hợp lệ
- **UC-17:** Cho crawler đọc sitemap, HTML và JSON-LD
- **UC-18:** Xuất canonical, hreflang, Open Graph và schema.org
