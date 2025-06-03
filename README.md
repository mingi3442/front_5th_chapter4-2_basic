# 바닐라 JS 프로젝트 성능 개선

- url: https://front-5th-chapter4-2-basic-liard.vercel.app/

# 성능 개선 보고서

## 1. 전체 성능 개선 결과
### Before

<img width="1800" alt="before-page-speed" src="https://github.com/user-attachments/assets/10af2733-7501-4bac-85ef-fa6b8d18953a" />

<img width="1800" alt="before" src="https://github.com/user-attachments/assets/067118f4-298e-4f86-8ece-a2e23d0c1d61" />

### After

<img width="1800" alt="after-page-speed" src="https://github.com/user-attachments/assets/b37d6022-1256-47db-b094-31af8326ccbb" />

<img width="1800" alt="after" src="https://github.com/user-attachments/assets/6e7c0fe1-e249-4c6b-9760-33a3e5380da2" />

### 결과

| 메트릭             | 개선 전 | 개선 후   | 개선도                 |
| ------------------ | ------- | --------- | ---------------------- |
| **Performance**    | 72%     | **95%**   | **+23%**               |
| **Accessibility**  | 82%     | **94%**   | **+12%**               |
| **Best Practices** | 75%     | **96%**   | **+21%**               |
| **SEO**            | 82%     | **91%**   | **+9%**                |
| **LCP**            | 14.56s  | **2.35s** | **-12.21s (83% 개선)** |

---

## 2. 개선 이유 및 방법

### 2-1. WebP 포맷 도입

#### 개선 이유

- **문제점**: JPEG 포맷만 사용하여 Hero 이미지가 1.08MB로 과도하게 큼
- **성능 영향**: LCP 14.56초 지연의 주원인
- **개선 필요성**: WebP 포맷 사용 시 67-77% 압축률 향상 가능

#### 개선 방법

```html
<!-- Before -->
<img src="images/Hero_Desktop.jpg" alt="VR Headsets" />

<!-- After -->
<picture>
  <source srcset="images/Hero_Desktop.webp" type="image/webp" />
  <source srcset="images/Hero_Desktop.jpg" type="image/jpeg" />
  <img src="images/Hero_Desktop.jpg" alt="VR Headsets" />
</picture>
```

**WebP 변환 작업:**

- 모든 이미지를 WebP 포맷으로 변환
- Picture 태그로 WebP 우선 제공, JPEG 폴백 지원
- 브라우저 호환성 확보

---

### 2-2. 반응형 이미지 구현

#### 개선 이유

- **문제점**: 모든 디바이스에서 동일한 큰 이미지(1440px) 로드
- **성능 영향**: 모바일에서 불필요하게 큰 파일 다운로드
- **개선 필요성**: 디바이스별 적절한 크기 제공으로 로딩 시간 단축

#### 개선 방법

```html
<picture>
  <!-- Desktop (1440px) -->
  <source
    media="(min-width: 961px)"
    srcset="images/Hero_Desktop.webp"
    type="image/webp" />

  <!-- Tablet (960px) -->
  <source
    media="(min-width: 577px) and (max-width: 960px)"
    srcset="images/Hero_Tablet.webp"
    type="image/webp" />

  <!-- Mobile (576px) -->
  <source
    media="(max-width: 576px)"
    srcset="images/Hero_Mobile.webp"
    type="image/webp" />

  <img src="images/Hero_Desktop.jpg" alt="VR Headsets" />
</picture>
```

**구현 전략:**

- 디바이스별 최적 해상도 이미지 제공
- 미디어 쿼리를 통한 조건부 로딩
- 모바일에서 89% 용량 절약 효과

---

### 2-3. 이미지 프리로드 적용

#### 개선 이유

- **문제점**: 중요한 히어로 이미지가 HTML 파싱 완료 후에 로드 시작
- **성능 영향**: LCP 지연 발생
- **개선 필요성**: 중요 리소스 우선 로딩으로 체감 성능 향상

#### 개선 방법

```html
<head>
  <link
    rel="preload"
    href="images/Hero_Desktop.webp"
    as="image"
    media="(min-width: 961px)" />
  <link
    rel="preload"
    href="images/Hero_Tablet.webp"
    as="image"
    media="(min-width: 577px) and (max-width: 960px)" />
  <link
    rel="preload"
    href="images/Hero_Mobile.webp"
    as="image"
    media="(max-width: 576px)" />
</head>
```

**적용 효과:**

- 이미지 로딩 우선순위 최상위로 설정
- HTML 파싱과 동시에 이미지 다운로드 시작
- LCP 2.35초 달성

---

### 2-4. Lazy Loading 적용

### 개선 이유

- **문제점**: 화면에 보이지 않는 제품 이미지들도 즉시 로드
- **성능 영향**: 초기 페이지 로딩 시간 증가 및 불필요한 대역폭 사용
- **개선 필요성**: 필요한 시점에만 로딩하여 초기 성능 최적화

### 개선 방법

```html
<img
  src="images/vr1.jpg"
  alt="Apple VR Headset"
  width="150"
  height="150"
  loading="lazy"
  decoding="async"
  sizes="(max-width: 576px) 140px, (max-width: 960px) 120px, 150px" />
```

**구현 세부사항:**

- 제품 이미지에 `loading="lazy"` 속성 적용
- `decoding="async"`로 이미지 디코딩 최적화
- `sizes` 속성으로 반응형 크기 지정

---

### 2-5. Google Tag Manager 지연 로딩

#### 개선 이유

- **문제점**: GTM이 페이지 초기 렌더링을 블로킹
- **성능 영향**: JavaScript 실행으로 인한 메인 스레드 블로킹
- **개선 필요성**: 추적 스크립트는 사용자 경험에 직접 영향 없으므로 지연 가능

#### 개선 방법

```javascript
function loadGTM() {
  (function (w, d, s, l, i) {
    w[l] = w[l] || [];
    w[l].push({ "gtm.start": new Date().getTime(), event: "gtm.js" });
    var f = d.getElementsByTagName(s)[0],
      j = d.createElement(s),
      dl = l != "dataLayer" ? "&l=" + l : "";
    j.async = true;
    j.src = "https://www.googletagmanager.com/gtm.js?id=" + i + dl;
    f.parentNode.insertBefore(j, f);
  })(window, document, "script", "dataLayer", "GTM-PKK35GL5");
}

// 3초 후 또는 사용자 상호작용 시 로딩
if (document.readyState === "complete") {
  setTimeout(loadGTM, 3000);
} else {
  window.addEventListener("load", () => setTimeout(loadGTM, 3000));
}

["click", "scroll", "keydown", "touchstart"].forEach((event) => {
  document.addEventListener(event, loadGTM, { once: true, passive: true });
});
```

**지연 로딩 전략:**

- 페이지 로드 완료 3초 후 자동 실행
- 사용자 상호작용 시 즉시 실행
- 초기 렌더링 블로킹 완전 제거

---

### 2-6. Cookie Consent 지연 로딩

#### 개선 이유

- **문제점**: Cookie Consent 라이브러리가 초기 로딩 블로킹
- **성능 영향**: 불필요한 초기 JavaScript 번들 크기 증가
- **개선 필요성**: 사용자가 페이지 탐색 후 표시해도 충분

#### 개선 방법

```javascript
window.addEventListener("load", function () {
  setTimeout(function () {
    const script = document.createElement("script");
    script.src =
      "//www.freeprivacypolicy.com/public/cookie-consent/4.1.0/cookie-consent.js";
    script.onload = function () {
      cookieconsent.run({
        notice_banner_type: "simple",
        consent_type: "express",
        palette: "light",
        language: "en",
        page_load_consent_levels: ["strictly-necessary"],
        notice_banner_reject_button_hide: false,
        preferences_center_close_button_hide: false,
        page_refresh_confirmation_buttons: false,
        website_name: "Performance Course",
      });
    };
    document.head.appendChild(script);
  }, 5000);
});
```

**동적 로딩 구현:**

- 5초 후 스크립트 동적 생성 및 로딩
- 초기 JavaScript 번들에서 완전 분리
- 25KB 초기 로딩 크기 절약

---

### 2-7. 이미지 크기 명시 및 Aspect Ratio

#### 개선 이유

- **문제점**: 이미지 로딩 전까지 크기를 알 수 없어 레이아웃 시프트 발생
- **성능 영향**: CLS(Cumulative Layout Shift) 점수 저하
- **개선 필요성**: 안정적인 레이아웃 제공으로 사용자 경험 향상

#### 개선 방법

```html
<!-- 히어로 이미지 크기 명시 -->
<img
  src="images/Hero_Desktop.jpg"
  alt="VR Headsets"
  width="1440"
  height="600"
  style="aspect-ratio: 1440/600; object-fit: cover"
  fetchpriority="high" />

<!-- 제품 이미지 정사각형 비율 -->
<img
  src="images/vr1.jpg"
  alt="Apple VR Headset"
  width="150"
  height="150"
  style="aspect-ratio: 1/1; object-fit: contain" />
```

**CSS 추가 최적화:**

```css
.hero img {
  aspect-ratio: 1440/600;
  object-fit: cover;
}

.product img {
  aspect-ratio: 1/1;
  object-fit: contain;
}
```

---

### 2-8. DNS 프리페치 및 폰트 최적화

#### 개선 이유

- **문제점**: 외부 리소스 DNS 조회 및 폰트 로딩으로 인한 지연
- **성능 영향**: 네트워크 레이턴시 증가, FOIT 발생
- **개선 필요성**: 연결 시간 단축 및 폰트 로딩 최적화

#### 개선 방법

```html
<!-- DNS 프리페치 -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com" />
<link rel="dns-prefetch" href="https://www.googletagmanager.com" />
<link rel="dns-prefetch" href="https://www.freeprivacypolicy.com" />

<!-- 폰트 최적화 -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Heebo:wght@300;400;600;700&display=swap"
  rel="stylesheet" />
```

**최적화 기법:**

- DNS 사전 해결로 연결 지연 제거
- `display=swap`으로 FOIT 방지
- 폰트 로딩 중에도 시스템 폰트로 텍스트 표시

---

### 2-9. 접근성 개선

#### 개선 이유

- **문제점**: 부정확한 alt 텍스트, 폼 접근성 부족
- **성능 영향**: 스크린 리더 사용자 배제, 웹 표준 미준수
- **개선 필요성**: 모든 사용자를 위한 포용적 웹 구현

#### 개선 방법

```html
<!-- 의미있는 alt 텍스트 -->
<img
  src="images/vr1.jpg"
  alt="Apple VR Headset with premium blue strap design - Advanced virtual reality device featuring cutting-edge optics and ergonomic comfort" />

<!-- 폼 접근성 강화 -->
<form method="POST" action="#" autocomplete="on">
  <input
    type="email"
    name="email"
    placeholder="Your email..."
    required
    autocomplete="email"
    pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
    aria-label="Email address for newsletter subscription" />
  <button type="submit">Subscribe</button>
</form>

<!-- 링크 보안 -->
<a href="#" rel="noopener noreferrer">External Link</a>
```

---

## 3. 개선 후 향상된 지표

### 3-1 Lighthouse 성능 점수

| 메트릭             | 개선 전 | 개선 후 | 개선율 |
| ------------------ | ------- | ------- | ------ |
| **Performance**    | 72%     | **95%** | +31.9% |
| **Accessibility**  | 82%     | **94%** | +14.6% |
| **Best Practices** | 75%     | **96%** | +28.0% |
| **SEO**            | 82%     | **91%** | +11.0% |

### 3-2 Core Web Vitals 개선

| 메트릭  | Google 기준 | 개선 전 | 개선 후    | 상태 |
| ------- | ----------- | ------- | ---------- | ---- |
| **LCP** | < 2.5초     | 14.56초 | **2.35초** | Good |
| **INP** | < 200ms     | N/A     | N/A        | Good |
| **CLS** | < 0.1       | 0.011   | 0.031      | Good |

### 3-3 파일 크기 최적화 결과

| 파일명       | JPEG 크기       | WebP 크기     | 압축률         |
| ------------ | --------------- | ------------- | -------------- |
| Hero_Desktop | 1,079,881 bytes | 351,392 bytes | **67.5% 감소** |
| Hero_Tablet  | 787,941 bytes   | 238,980 bytes | **69.7% 감소** |
| Hero_Mobile  | 414,700 bytes   | 121,588 bytes | **70.7% 감소** |
| VR1          | 53,702 bytes    | 12,496 bytes  | **76.7% 감소** |
| VR2          | 90,633 bytes    | 24,236 bytes  | **73.3% 감소** |
| VR3          | 76,401 bytes    | 17,336 bytes  | **77.3% 감소** |
| **총계**     | **2.5MB**       | **0.8MB**     | **71.2% 감소** |
