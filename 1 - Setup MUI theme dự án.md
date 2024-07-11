# Cài đặt MUI theme dự án

Material UI là một thư viện mã nguồn mở hỗ trợ cung cấp cho các developer những bộ UI components và configs dựng sẵn theo nguyên tắc Material Design của Google. Nó giúp các developer làm việc với UI nhanh hơn, tiện lợi hơn.

## Cài đặt MUI

- Sử dụng lệnh `yarn add @mui/material @emotion/react @emotion/styled` để cài đặt MUI và các gói liên quan cần dùng.

## Cấu hình theme cơ bản

- Trong file `App.tsx`:
  - Dùng `ThemeProvider` bao ngoài cùng để áp dụng theme cho toàn bộ ứng dụng.
  - Tạo `theme` với `createTheme` để tùy chỉnh theme.
  - Dùng `CssBaseline` hỗ trợ cấu hình các style mặc định

```jsx
import {  CssBaseline,ThemeProvider, createTheme } from '@mui/material';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
});

function App(){

...

 return (
    <ThemeProvider theme={theme}>
      <div className="App">
      ...
      </div>
      <CssBaseline />
    </ThemeProvider>
  );
}

export default App;
```

- Trong folder `theme`:

  - Tải tài nguyên theme từ [Theme](https://vitejs.dev/) và bỏ những file cấu hình theme có sẵn vào folder.
  - Tạo hook `useTheme` trong folder theme để tách biệt logic cấu hình theme và nhiều logic về theme khác.

- Trong file `App.tsx`:
  - Import `theme` return từ `useTheme` vào lại như ban đầu.

```jsx
// theme/index.ts
import { createTheme } from "@mui/material/styles";

export const useTheme = () => {
  const theme = createTheme({
    palette: {
      primary: {
        main: "#1976d2",
      },
      secondary: {
        main: "#dc004e",
      },
    },
  });
  return theme;
};
```

```jsx
import { useTheme } from './theme';
import { CssBaseline, ThemeProvider } from '@mui/material';

function App(){
 const theme = useTheme();
...

 return (
    <ThemeProvider theme={theme}>
      <div className="App">
      ...
      </div>
      <CssBaseline />
    </ThemeProvider>
  );
}

export default App;
```

## Cấu hình theme nâng cao

- Trong file `theme/index.tsx`:
  - Khởi tạo hook `useTheme`.
  - Tạo biến tạm mặc định `activeTheme` và `activeMode`.
  - Tạo `lightThemeOptions` dựa trên `LightThemeColors` và `darkThemeOptions` dựa trên `DarkThemeColors` đã được cấu hình sẵn trong folder theme.
  - Tạo `defaultTheme`, Dựa vào `activeTheme` kiểm tra điều kiện để xác định `defaultTheme`.
  - Thêm `defaultTheme` vào cấu hình `createTheme` trong `theme`.

```jsx
import { ThemeMode } from "@/constants/theme";
import DarkThemeColors from "./DarkThemeColors";
import { baseDarkTheme, baselightTheme } from "./DefaultColors";
import LightThemeColors from "./LightThemeColors";

export const useTheme = () => {
  const activeTheme = "BLUE_THEME";
  const activeMode = "light";
  const lightThemeOptions = LightThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const darkThemeOptions = DarkThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const defaultTheme =
    activeMode === ThemeMode.DARK ? baseDarkTheme : baselightTheme;

  const theme = createTheme(
    _.merge({}, defaultTheme, {
      direction: "ltr",
    })
  );

  return theme;
};
```

- Tạo `defaultShadow`, `themeSelect` và `baseMode`. Dựa vào `activeMode` để tùy chỉnh cấu hình theme.
- Dùng `Typography` cấu hình font size, font weight,...
- Dùng `components` để cấu hình style cho components từ folder theme dựng sẵn.

```jsx
import { darkshadows, shadows } from "./Shadows";
import Typography from "./Typography";
import * as locales from "@mui/material/locale";
import components from "./Components";

export const useTheme = () => {
  const activeTheme = "BLUE_THEME";
  const activeMode = "light";
  const borderRadius = "4px";

  const lightThemeOptions = LightThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const darkThemeOptions = DarkThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const defaultTheme =
    activeMode === ThemeMode.DARK ? baseDarkTheme : baselightTheme;
  const defaultShadow = activeMode === ThemeMode.DARK ? darkshadows : shadows;
  const themeSelect =
    activeMode === ThemeMode.DARK ? darkThemeOptions : lightThemeOptions;

  const baseMode = {
    palette: {
      mode: activeMode,
    },
    shape: {
      borderRadius: borderRadius,
    },
    shadows: defaultShadow,
    typography: Typography,
  };

  const theme = createTheme(
    _.merge({}, baseMode, defaultTheme, locales, themeSelect, {
      direction: "ltr",
    })
  );
  theme.components = components(theme);

  return theme;
};
```

## Cấu hình redux và thay đổi theme với redux

### 1. Cấu hình redux

- Sử dụng lệnh `yarn add react-redux @reduxjs/toolkit` để cài đặt redux vào dự án.
- Trong folder `store`:
  - Tạo file `index.ts` để cấu hình store cho dự án.
  - Dùng `configureStore` để khởi tạo store.
  - Export các type `AppState` và `AppDispatch` để dùng cho các trường hợp định nghĩa type cho `selector` và `dispatch` trong dự án.
  - Export 2 custom hook `useAppDispatch` và `useAppSelector` để dùng chung đã bao gồm setup có type, thay vì dùng `useDispatch` và `useSelector` mặc định.

```jsx
import { configureStore } from '@reduxjs/toolkit';
import {
  useDispatch,
  useSelector
} from 'react-redux';

export const store = configureStore({
  reducer: {
  },
});

// https://redux-toolkit.js.org/tutorials/typescript#define-typed-hooks
// Infer the `AppState` and `AppDispatch` types from the store itself
export type AppState = ReturnType<typeof store.getState>
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = useDispatch.withTypes<AppDispatch>()
export const useAppSelector = useSelector.withTypes<AppState>()

export default store;
```

- Trong file `main.tsx`:
  - Dùng `Provider` bao ngoài `App` để sử dụng các tính năng của Redux.
  - Import `store` vào `main.tsx` và gán `store` vừa tạo vào store của `Provider` để làm store của dự án.

```jsx
import { Provider } from 'react-redux';
import store from './store/index.tsx';

ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <Provider store={store}>
          <App />
       </Provider>,
    </React.StrictMode>,
);
```

### 2. Thay đổi theme với redux

- Trong folder `constants`:

  - Tạo file `theme.ts` và thêm enum `ThemeMode` và `ThemeColor` để định nghĩa theme và mode.

- Trong folder `store`:
  - Tạo folder `customizer`.
  - Tạo file `customer/CustomizerSlice.ts`.
  - Định nghĩa type `StateType` để định nghĩa type cho state của `customizer` slice.
  - Thêm `initialState` là state các giá trị ban đầu cấu hình cơ bản của theme có sẵn.

```jsx
export enum ThemeMode {
    LIGHT = 'light',
    DARK = 'dark',
}
export enum ThemeColor {
    BLUE_THEME = 'blue',
    GREEN_THEME = 'green',
    BLACK_THEME = 'black',
    PURPLE_THEME = 'purple',
    ORANGE_THEME = 'orange',
    CYAN_THEME = 'cyan',
    AQUA_THEME = 'aqua',
}
```

```jsx
import { createSlice } from "@reduxjs/toolkit";
import { ThemeColor, ThemeMode } from "../../constants";

interface StateType {
  activeMode?: string;
  activeTheme?: string;
  SidebarWidth?: number;
  MiniSidebarWidth?: number;
  TopbarHeight?: number;
  isCollapse?: boolean;
  isLayout?: string;
  isSidebarHover?: boolean;
  isMobileSidebar?: boolean;
  isHorizontal?: boolean;
  isLanguage?: string;
  isCardShadow?: boolean;
  borderRadius?: number;
}

const initialState: StateType = {
  activeMode: ThemeMode.LIGHT,
  activeTheme: ThemeColor.BLUE_THEME,
  SidebarWidth: 270,
  MiniSidebarWidth: 87,
  TopbarHeight: 70,
  isLayout: "boxed",
  isCollapse: false,
  isSidebarHover: false,
  isMobileSidebar: false,
  isHorizontal: false,
  isLanguage: "en",
  isCardShadow: true,
  borderRadius: 7,
};
```

- Khởi tạo `CustomizerSlice` với `createSlice` của `@reduxjs/toolkit` để tạo slice.
- Thêm các actions liên quan như `setTheme`, `setDarkMode`,... để thay đổi theme.
- Export các actions để bên ngoài có thể gọi và xử lý.

```jsx
import { ThemeColor, ThemeMode } from '@/constants/theme';
import { createSlice } from '@reduxjs/toolkit';

interface StateType {
...
}
const initialState: StateType = {
...
}

...

export const CustomizerSlice = createSlice({
  name: 'customizer',
  initialState,
  reducers: {
    setTheme: (state: StateType, action) => {
      state.activeTheme = action.payload;
    },
    setDarkMode: (state: StateType, action) => {
      state.activeMode = action.payload;
    },
    setLanguage: (state: StateType, action) => {
      state.isLanguage = action.payload;
    },
    setCardShadow: (state: StateType, action) => {
      state.isCardShadow = action.payload;
    },
    toggleSidebar: (state) => {
      state.isCollapse = !state.isCollapse;
    },
    hoverSidebar: (state: StateType, action) => {
      state.isSidebarHover = action.payload;
    },
    toggleMobileSidebar: (state) => {
      state.isMobileSidebar = !state.isMobileSidebar;
    },
    toggleLayout: (state: StateType, action) => {
      state.isLayout = action.payload;
    },
    toggleHorizontal: (state: StateType, action) => {
      state.isHorizontal = action.payload;
    },
    setBorderRadius: (state: StateType, action) => {
      state.borderRadius = action.payload;
    },
  },
});

export const {
  setTheme,
  setDarkMode,
  toggleSidebar,
  hoverSidebar,
  toggleMobileSidebar,
  toggleLayout,
  setBorderRadius,
  toggleHorizontal,
  setLanguage,
  setCardShadow,
} = CustomizerSlice.actions;

export default CustomizerSlice.reducer;
```

- Trong file `store/index.ts`:
  - Import `CustomizerReducer` vào `reducer`

```jsx
import CustomizerReducer from './customizer/CustomizerSlice';

export const store = configureStore({
  reducer: {
    customizer: CustomizerReducer,
  },
});

...

export default store;
```

- Trong hook `useTheme`:
- Loại bỏ những biến mặc định `activeTheme`, `activeMode`, `borderRadius`.
- Lấy ra `customizer` từ store các cấu hình theme `activeTheme`,`activeMode` và `borderRadius` đã khai báo ở `CustomizerSlice`.

```jsx
import { useSelector } from "react-redux";

export const useTheme = () => {
  const customizer = useSelector((state) => state.customizer);
  const { activeTheme, activeMode, borderRadius } = customizer || {};
  const lightThemeOptions = LightThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const darkThemeOptions = DarkThemeColors.find(
    (theme) => theme.name === activeTheme
  );
  const defaultTheme =
    activeMode === ThemeMode.DARK ? baseDarkTheme : baselightTheme;
  const defaultShadow = activeMode === ThemeMode.DARK ? darkshadows : shadows;
  const themeSelect =
    activeMode === ThemeMode.DARK ? darkThemeOptions : lightThemeOptions;

  const baseMode = {
    palette: {
      mode: activeMode,
    },
    shape: {
      borderRadius: borderRadius,
    },
    shadows: defaultShadow,
    typography: Typography,
  };

  const theme = createTheme(
    _.merge({}, baseMode, defaultTheme, locales, themeSelect, {
      direction: "ltr",
    })
  );
  theme.components = components(theme);

  return theme;
};
```
