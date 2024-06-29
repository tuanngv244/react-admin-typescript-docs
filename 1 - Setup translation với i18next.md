# Cài đặt translation với i18next

i18next là một framework hỗ trợ cho phép thực hiện chức năng dịch đa ngôn ngữ.
Trong thực tế khi một dự án ra đời sẽ thường có tính năng dịch đa ngôn ngữ hỗ trợ tiếp cận nội dung tốt hơn cho người dùng.

## Cài đặt i18next

- Sử dụng lệnh `yarn add i18next` và `yarn add react-i18next` để cài đặt i18next vào dự án.

## Cấu hình cơ bản

Để cấu hình i18next vào dự án bạn cần:

### 1. Khai báo constant cần thiết:

- Tạo một enum `Elanguage` trong `constant/i18n.ts` để định nghĩa ngôn ngữ có trong dự án.
- Thêm một key `language` trong `constant/storage.ts` để lưu active ngôn ngữ trong storage.

```jsx
/// constant/i18n.ts
export enum ELanguages {
    EN = 'en',
    VI = 'vi',
}
```

```jsx
/// constant/storage.ts
export const STORAGE = {
  language: "cfd_language",
};
```

### 2. Khởi tạo file cấu hình i18next:

- Tại folder `languages` tạo 1 file `index.ts`.
- Khai báo `languageDetector` cấu hình hỗ trợ phát hiện ngôn ngữ.
- Gọi i18next dùng hàm `use` để thêm cấu hình và `init` để khởi tạo trình dịch.

```jsx
/// languages/i18n.ts
import { initReactI18next } from "react-i18next";
import i18next, { LanguageDetectorAsyncModule } from "i18next";

import en from "./locales/en.json";
import vi from "./locales/vi.json";
import { STORAGE } from "@/constants/storage";
import { ELanguages } from "@/constants/i18n";

const languageDetector: LanguageDetectorAsyncModule = {
  type: "languageDetector",
  async: true,
  detect: (cb) => cb("vi"),
  init: () => {},
  cacheUserLanguage: () => {},
};

const resources = {
  vi: { translation: vi },
  en: { translation: en },
};

i18next.use(languageDetector).use(initReactI18next).init({
  fallbackLng: ELanguages.VI,
  debug: false,
  resources: resources,
  compatibilityJSON: "v3",
});
```

- Khai báo hàm `initLanguage` để khởi tạo ngôn ngữ ban đầu khi chạy dự án.
- Khai báo hàm `setLanguge` để set lại ngôn ngữ mới khi người dùng đổi ngôn ngữ và lưu ngôn ngữ mới vào `localStorage`.

```jsx
/// languages/i18n.ts

...

const initLanguage = () => {
  const initialLanguage =
    localStorage.getItem(STORAGE.language) || ELanguages.EN;
  i18next.changeLanguage(initialLanguage);
};

const setLanguage = (value: ELanguages) => {
  localStorage.setItem(STORAGE.language, value);
  i18next.changeLanguage(value);
};

...

```

- Khai báo biến `languageOptions` để sử dụng ở nơi khác cần dùng đến.
- Xuất biến `I18nApp` để import vào `App.tsx` khởi tạo i18next.

```jsx

...

const languageOptions = [
  {
    label: "Vietnamese",
    value: ELanguages.VI,
  },
  {
    label: "English",
    value: ELanguages.EN,
  },
];

export { languageOptions, initLanguage, setLanguage };

const I18nApp = i18next;
export default I18nApp;
```

### 3. Import vào App để khởi chạy i18next:

- Import `i18n.ts` vào `App.tsx`.
- Sử dụng useEffect vào gọi `initLanguage` để khởi chạy i18next.
- Sử dụng `setLanguage` để thay đổi ngôn ngữ.

```jsx
import "@/languages/index";
import { initLanguage, setLanguage } from "@/languages/index";
import { useTranslation } from "react-i18next";
import { useEffect } from "react";
import { ELanguages } from "./constants/i18n";

function App() {
  const { t } = useTranslation();

  const _onLanguageChange = (language: ELanguages) => setLanguage(language);

  useEffect(() => {
    initLanguage();
  }, []);

  return (
    <div>
      {t("COMMON.content")}
      {t("COMMON.save")}
      {t("COMMON.cancel")}
      <button onClick={() => _onLanguageChange(ELanguages.EN)}>English</button>
      <button onClick={() => _onLanguageChange(ELanguages.VI)}>
        Vietnamese
      </button>
    </div>
  );
}
```
