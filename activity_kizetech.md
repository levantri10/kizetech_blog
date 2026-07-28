# SƠ ĐỒ HOẠT ĐỘNG — HỆ THỐNG XUẤT BẢN BLOG KIZETECH

> **Nguyên tắc:** Mọi giai đoạn là blocking gate — lỗi ảnh, nội dung, audit, build, deploy hoặc xác minh đều dừng pipeline.

## Tổng quan Swimlane

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#F8FAFC','primaryBorderColor':'#94A3B8','lineColor':'#475569','secondaryColor':'#FFFFFF','tertiaryColor':'#EFF6FF'}}}%%
flowchart LR
    subgraph AUTHOR["TÁC GIẢ / BIÊN TẬP VIÊN"]
        A_START[("● START")]
        A_SUBMIT["Gửi bản thảo / ý tưởng<br/>và yêu cầu xuất bản"]
        A_ADDIMG["Đặt ảnh vào<br/>imge(nopush)/"]
        A_ERR_IMG["DỪNG: báo thiếu ảnh / file lỗi"]
        A_ERR_UPLOAD["DỪNG: nêu chính xác filename thất bại"]
        A_REVISE2["Sửa bản thảo / frontmatter<br/>và gửi lại"]
        A_REPORT2["Báo field/checklist lỗi + trích đoạn"]
        A_REPORT3["DỪNG: liệt kê rule + dòng vi phạm<br/>không tự rewrite"]
        A_REVISE3["Tự sửa Markdown<br/>rồi chạy lại Gate 3"]
        A_ERR_BUILD["DỪNG: sửa lỗi build / sitemap; không commit"]
        A_ERR_DEPLOY["DỪNG: debug deploy / cache / sitemap"]
        A_END[("● END")]
    end

    subgraph ORCHESTRATOR["ORCHESTRATOR / STAGE 1 & 4"]
        O_READSKILL["Đọc master skill và 4 stage contract"]
        O_SCAN["Stage 1: quét file ảnh hợp lệ<br/>jpg/jpeg/png/webp/gif/avif/svg"]
        O_HASIMG{"Có ít nhất<br/>1 ảnh?"}
        O_UPLOAD["Chạy upload-from-nopush.mjs<br/>--category --slug"]
        O_UPLOAD_OK{"Tất cả upload<br/>thành công?"}
        O_ASSETS["Nhận assets[] JSON; chuyển file gốc vào<br/>_processed/{YYYY-MM-DD}/"]
        O_NORMALIZE["Sinh / chuẩn hóa Markdown + frontmatter<br/>gắn URL Cloudinary"]
        O_TARGET["Xác định target file, blog URL; audit gitignore, secrets, identity"]
        O_SITEMAP["npm run sitemap<br/>ghi public/sitemap.xml"]
        O_BUILD["npm run build<br/>Astro SSG → dist/"]
        O_BUILD_OK{"Build thành công +<br/>dist/sitemap.xml có hreflang?"}
        O_COMMIT["Stage target + sitemap; commit và push"]
        O_RESOLVE_URLS["Tìm các URL EN/VI cùng translationGroup"]
        O_REPORTS["Ghi và đọc báo cáo:<br/>index-report.json + indexnow-report.json"]
    end

    subgraph CLOUD["CLOUDINARY"]
        C_UPLOAD["Upload vào<br/>post/{category}/{slug}/<br/>trả URL f_auto,q_auto"]
    end

    subgraph GATE["STAGE 2 + STAGE 3 / QUALITY GATES"]
        G2["Stage 2 — Content Quality Gate<br/>frontmatter + 4-section framework + checklist A/B/C"]
        G2_PASS{"Tất cả check<br/>Stage 2 đạt?"}
        G3["Stage 3 — Final Compliance Audit<br/>7 editorial + 4 mechanical/safety rules"]
        G3_PASS{"Đủ 11/11<br/>tiêu chí?"}
    end

    subgraph DEPLOY["GITHUB + VERCEL"]
        D_GITHUB["GitHub nhận commit<br/>Vercel chạy npm install + npm run build<br/>và phát hành dist/"]
        D_LIVE{"Sitemap production có slug,<br/>lastmod, hreflang đúng?"}
    end

    subgraph INDEX["GOOGLE + INDEXNOW"]
        I_NOTIFY["Submit từng URL đã đổi:<br/>Google Indexing API + IndexNow"]
    end
```

## Chi tiết luồng Stage 1 — Image Upload

```mermaid
flowchart TD
    S1_START(["● START"])
    S1_SUBMIT["Tác giả gửi bản thảo"]
    S1_READ["Orchestrator đọc master skill"]
    S1_ADDIMG["Tác giả đặt ảnh vào imge(nopush)/"]
    S1_SCAN["Quét file ảnh hợp lệ<br/>(jpg/jpeg/png/webp/gif/avif/svg)"]
    S1_CHECK{"Có ít nhất<br/>1 ảnh?"}
    S1_ERR1["❌ DỪNG: báo thiếu ảnh / file lỗi"]
    S1_UPLOAD["Chạy upload-from-nopush.mjs<br/>--category --slug"]
    S1_CLOUD["Cloudinary upload<br/>post/{category}/{slug}/<br/>f_auto, q_auto"]
    S1_OK{"Tất cả upload<br/>thành công?"}
    S1_ERR2["❌ DỪNG: nêu chính xác filename thất bại"]
    S1_ASSETS["Nhận assets[] JSON<br/>Di chuyển file gốc → _processed/{YYYY-MM-DD}/"]
    S1_NORM["Sinh / chuẩn hóa Markdown + frontmatter<br/>gắn URL Cloudinary"]

    S1_START --> S1_SUBMIT
    S1_SUBMIT --> S1_READ
    S1_READ --> S1_SCAN
    S1_ADDIMG -.->|"bổ sung ảnh"| S1_SCAN
    S1_SCAN --> S1_CHECK
    S1_CHECK -->|"Có ✅"| S1_UPLOAD
    S1_CHECK -->|"Không ❌"| S1_ERR1
    S1_ERR1 -.->|"Bổ sung ảnh"| S1_ADDIMG
    S1_UPLOAD -->|"API upload"| S1_CLOUD
    S1_CLOUD -->|"assets[] / error"| S1_OK
    S1_OK -->|"Có ✅"| S1_ASSETS
    S1_OK -->|"Không ❌"| S1_ERR2
    S1_ERR2 -.->|"Thay ảnh / thử lại"| S1_ADDIMG
    S1_ASSETS --> S1_NORM
    S1_NORM --> S1_END(["→ Gate 2"])

    style S1_ERR1 fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style S1_ERR2 fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style S1_END fill:#F0FDF4,stroke:#16A34A,color:#166534
```

## Chi tiết luồng Gate 2 & Gate 3

```mermaid
flowchart TD
    G2_START(["→ Từ Stage 1"])
    G2_CHECK["Stage 2 — Content Quality Gate<br/>frontmatter + 4-section framework + checklist A/B/C"]
    G2_DECISION{"Tất cả check<br/>Stage 2 đạt?"}
    G2_REPORT["❌ Báo field/checklist lỗi + trích đoạn"]
    G2_REVISE["Tác giả sửa bản thảo / frontmatter"]
    G3_CHECK["Stage 3 — Final Compliance Audit<br/>7 editorial + 4 mechanical/safety rules"]
    G3_DECISION{"Đủ 11/11<br/>tiêu chí?"}
    G3_REPORT["❌ DỪNG: liệt kê rule + dòng vi phạm<br/>không tự rewrite"]
    G3_REVISE["Tự sửa Markdown rồi chạy lại Gate 3"]
    G3_PASS["✅ PASS — sang Stage 4 (deploy-index)"]

    G2_START --> G2_CHECK
    G2_CHECK --> G2_DECISION
    G2_DECISION -->|"Có ✅"| G3_CHECK
    G2_DECISION -->|"Không ❌"| G2_REPORT
    G2_REPORT --> G2_REVISE
    G2_REVISE -.->|"chạy lại Stage 2"| G2_CHECK
    G3_CHECK --> G3_DECISION
    G3_DECISION -->|"Có ✅"| G3_PASS
    G3_DECISION -->|"Không ❌"| G3_REPORT
    G3_REPORT --> G3_REVISE
    G3_REVISE -.->|"chạy lại Gate 3"| G3_CHECK

    style G2_REPORT fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style G3_REPORT fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style G3_PASS fill:#F0FDF4,stroke:#16A34A,color:#166534
```

## Chi tiết luồng Stage 4 — Deploy & Index

```mermaid
flowchart TD
    S4_START(["→ Từ Gate 3 PASS"])
    S4_TARGET["Xác định target file, blog URL<br/>audit .gitignore, secrets, identity"]
    S4_SITEMAP["npm run sitemap<br/>ghi public/sitemap.xml"]
    S4_BUILD["npm run build<br/>Astro SSG → dist/"]
    S4_BUILD_OK{"Build thành công +<br/>dist/sitemap.xml có hreflang?"}
    S4_ERR_BUILD["❌ DỪNG: sửa lỗi build / sitemap<br/>không commit"]
    S4_COMMIT["Stage target + sitemap<br/>commit và push"]
    S4_GITHUB["GitHub nhận commit<br/>Vercel chạy npm install + npm run build<br/>phát hành dist/"]
    S4_LIVE{"Sitemap production có slug,<br/>lastmod, hreflang đúng?"}
    S4_ERR_DEPLOY["❌ DỪNG: debug deploy / cache / sitemap"]
    S4_RESOLVE["Tìm các URL EN/VI cùng translationGroup"]
    S4_NOTIFY["Submit từng URL đã đổi<br/>Google Indexing API + IndexNow"]
    S4_REPORTS["Ghi và đọc báo cáo<br/>index-report.json + indexnow-report.json"]
    S4_END(["● END"])

    S4_START --> S4_TARGET
    S4_TARGET --> S4_SITEMAP
    S4_SITEMAP --> S4_BUILD
    S4_BUILD --> S4_BUILD_OK
    S4_BUILD_OK -->|"Có ✅"| S4_COMMIT
    S4_BUILD_OK -->|"Không ❌"| S4_ERR_BUILD
    S4_ERR_BUILD -.->|"sửa và build lại"| S4_SITEMAP
    S4_COMMIT -->|"git push"| S4_GITHUB
    S4_GITHUB --> S4_LIVE
    S4_LIVE -->|"Có ✅"| S4_RESOLVE
    S4_LIVE -->|"Không ❌"| S4_ERR_DEPLOY
    S4_ERR_DEPLOY -.->|"xác minh lại"| S4_LIVE
    S4_RESOLVE --> S4_NOTIFY
    S4_NOTIFY -->|"responses"| S4_REPORTS
    S4_REPORTS --> S4_END

    style S4_ERR_BUILD fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style S4_ERR_DEPLOY fill:#FFF1F2,stroke:#E11D48,color:#9F1239
    style S4_END fill:#172033,stroke:#172033,color:#FFFFFF
```

## Lưu ý thứ tự bắt buộc ở Stage 4

1. **sitemap trước build** — `npm run sitemap` phải chạy TRƯỚC `npm run build`
2. **chỉ commit/push sau khi build pass** — verify `dist/sitemap.xml` có `xhtml:link` trước
3. **chỉ submit các URL cụ thể sau khi production sitemap đã đúng** — sitemap live trên Vercel phải đúng trước

## Bảng trạng thái Gate

| Gate | Điều kiện PASS | Action khi FAIL |
|------|----------------|-----------------|
| **Stage 1** (Image) | ≥1 ảnh hợp lệ, upload thành công | DỪNG — báo lỗi cụ thể |
| **Gate 2** (Content) | frontmatter OK + 4-section + checklist A/B/C | DỪNG — trích field lỗi, tác giả sửa |
| **Gate 3** (Final) | 11/11 tiêu chí Zero-Slop | DỪNG — liệt kê rule + dòng, không auto-rewrite |
| **Stage 4** (Build) | Build thành công + sitemap có hreflang | DỪNG — sửa và build lại |
| **Stage 4** (Deploy) | Sitemap production đúng | DỪNG — debug deploy/cache |
