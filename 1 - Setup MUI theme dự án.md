# Cài đặt MUI theme dự án

Material UI là một thư viện mã nguồn mở hỗ trợ cung cấp cho các developer những bộ UI components và configs dựng sẵn theo nguyên tắc Material Design của Google. Nó giúp các developer làm việc với UI nhanh hơn, tiện lợi hơn.

## Cài đặt MUI

- Sử dụng lệnh `yarn add @mui/material @emotion/react @emotion/styled` để cài đặt MUI và các gói liên quan cần dùng.

## Cấu hình theme cơ bản

- Bọc `ThemeProvider` ngoài cùng trong file `App.tsx` để cung cấp context tài nguyên của theme trên toàn dự án.
- Thêm biến `theme` bằng function `createTheme` dùng để tạo theme cấu hình ban đầu. Material UI cung cấp function `createTheme` để cấu hình cho dự án như màu sắc, kiểu chữ, kích thước,...
- Thêm `CssBaseline` hỗ trợ cấu hình các style mặc định như loại bỏ margin, box-sizing, màu nền, màu scrollbar,...

(Thiện review)
- Trong file `App.tsx`:
  - Dùng `ThemeProvider` bao ngoài cùng để cung...
  - Dùng `createTheme`...

```jsx
import { ThemeProvider, createTheme } from '@mui/material';

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

- Tải tài nguyên theme và bỏ những file cấu hình theme có sẵn vào folder theme trong dự án.
- Nâng cao hơn chúng ta tạo hook `useTheme` trong folder theme để tiện cấu hình theme và nhiều logic về theme khác.
- Import biến `theme` từ `useTheme` vào lại `App.tsx` như ban đầu.

```jsx
// theme/index.ts
import { createTheme } from '@mui/material/styles';

export const useTheme = () => {
    const theme = createTheme(
     {
        palette: {
        primary: {
          main: '#1976d2',
        },
        secondary: {
          main: '#dc004e',
        },
      }
    );

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

- Thêm biến dữ liệu tạm mặc định `activeTheme` và `activeMode`.
- Thêm cấu hình chế độ `light mode/dark mode` trong theme.
- Khai báo `lightThemeOptions` dựa trên `LightThemeColors` và `darkThemeOptions` dựa trên `DarkThemeColors` đã được cấu hình sẵn trong folder theme.
- Khai báo 2 biến tạm ban đầu `activeTheme` và `activeMode` nhằm mục đích để bật/tắt chế độ theme và light/dark mode trong tương lai.
- Dựa vào `activeTheme` và `activeMode` kiểm tra điều kiện để biết chính xác được `defaultTheme`.
- Thêm `defaultTheme` vào cấu hình `createTheme` ở biến theme.

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

- Dựa vào `activeTheme` và `activeMode` chúng ta tiếp tục cấu hình hoàn thiện theme với `defaultShadow`, `themeSelect` và `baseMode`.
- Thêm biến `baseMode` để thêm thông tin cấu hình style trong dự án.
- Thêm`Typography` cấu hình font size, font weight,...
- Thêm biến `defaultShadow` để cấu hình style shadow.
- Thêm biến`themeSelect` dùng để set lại theme options khi theme được thay đổi bởi user.
- Thêm `components` dùng để cấu hình style cho components từ folder theme dựng sẵn.

```jsx
import { darkshadows, shadows } from "./Shadows";
import Typography from "./Typography";
import * as locales from "@mui/material/locale";

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
- Thêm file `store/index.ts` để cấu hình store cho dự án.
- Sử dụng `configureStore` để khởi tạo store.
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

- Import `store` vừa tạo vào file `main.tsx`.
- Bọc `App` lại bằng `Provider` của `react-redux` để sử dụng các tính năng của Redux.
- Gán `store` vừa tạo vào store của `Provider` để làm store của dự án.

```jsx
import { Provider } from 'react-redux';
import store from './store/index.tsx';

ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
      <Provider store={store}>
          <App />
      </React.StrictMode>,
    </Provider>,
);
```

### 2. Thay đổi theme với redux

- Khởi tạo module `customizer` trong folder store. Tạo folder `customizer` và file `customer/CustomizerSlice.ts`.
- Thêm type `StateType` để định nghĩa type cho state của `CustomizerSlice`.
- Thêm `initialState` là state các giá trị ban đầu cấu hình cơ bản của theme có sẵn.

```jsx
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
  borderRadius?: number | any;
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

- Import `CustomizerReducer` vào `reducer` trong store.

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

- Tiếp theo thêm `constants/theme.ts` và thêm enum `ThemeMode` và `ThemeColor` để định nghĩa theme và mode.

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

- Tiếp tục trở lại `useTheme` sau khi đã có cấu hình theme với redux chúng ta loại bỏ những biến mặc định `activeTheme`, `activeMode`, `borderRadius`.
- Lấy ra `customizer` từ store các cấu hình theme mà chúng ta đã khai báo ở `CustomizerSlice` để xác định cấu hình theme theo tương tác của người dùng.

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
