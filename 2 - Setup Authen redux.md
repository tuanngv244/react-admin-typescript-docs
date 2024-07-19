# Setup Authen redux

Xây dựng luồng authentication với redux. Việc xây dựng luồng authentication (xác thực) với Redux trong một ứng dụng React giúp quản lý trạng thái xác thực của người dùng một cách hiệu quả. 

## Setup authen page và authen layout

- Trong folder `pages`:

  - Tạo `Login` page.
  - Tạo `Register` page.

- Trong folder `layouts`:

  - Tạo `AuthenLayout` dùng làm layout bao ngoài cho các authen route.

- Trong file `configs/routes`:
  - Setup route cho login page,register page.
  - Dùng `AuthenLayout` bao ngoài các route login/register.

```jsx
// Login/index.tsx
import React from "react";

const Login = () => {
  return <div>Login page</div>;
};

export default Login;
```

- Register page.

```jsx
// Register/index.tsx
import React from "react";

const Register = () => {
  return <div>Login page</div>;
};

export default Register;
```

- Authentication layout.

```jsx
// AuthenLayout/index.tsx
import ImgLogo from "@/assets/logo/logo.png";
import { Box, Grid } from "@mui/material";
import { Outlet } from "react-router-dom";

const AuthenLayout = () => {
  return (
    <Grid container spacing={0} sx={{ overflowX: "hidden" }}>
      <Grid
        item
        xs={12}
        sm={12}
        lg={7}
        xl={8}
        sx={{
          position: "relative",
          "&:before": {
            content: '""',
            background: "radial-gradient(#d2f1df, #d3d7fa, #bad8f4)",
            backgroundSize: "400% 400%",
            animation: "gradient 15s ease infinite",
            position: "absolute",
            height: "100%",
            width: "100%",
            opacity: "0.3",
          },
        }}
      >
        <Box position="relative">
          <Box
            alignItems="center"
            justifyContent="center"
            height={"100dvh"}
            sx={{
              display: {
                xs: "none",
                lg: "flex",
              },
            }}
          >
            <img
              src={ImgLogo}
              alt="bg"
              style={{
                width: "100%",
                maxWidth: "227px",
              }}
            />
          </Box>
        </Box>
      </Grid>
      <Grid
        item
        xs={12}
        sm={12}
        lg={5}
        xl={4}
        display="flex"
        justifyContent="center"
        alignItems="center"
      >
        <Box p={4}>
          <Outlet />
        </Box>
      </Grid>
    </Grid>
  );
};

export default AuthenLayout;
```

```jsx
// routes.ts
...

import AuthenLayout from '@/layouts/AuthenLayout';

...

const Login = withSuspenseLoader(lazy(() => import('../pages/Login')));
const Register = withSuspenseLoader(lazy(() => import('../pages/Register')));


const routes: RouteObject[] = [
  {
    path: Paths.ROOT,
    element: (
      <PrivateRoute
        redirectTo={Paths.AUTHENTICATION}
        element={<MenuLayout />}
      />
    ),
    children: [
      ...
    ],
  },
  {
     path: Paths.ROOT,
    element: <BlankLayout />,
    children: [
      {
        path: Paths.AUTHENTICATION,
        element: <AuthenLayout />,
        children: [
          {
            index: true,
            path: Paths.AUTHENTICATION,
            element: <Login />,
          },
            {
            index: true,
            path: Paths.AUTHENTICATION,
            element: <Register />,
          },
        ],
      },
      { path: Paths.STATUS_404, element: <Status404 /> },
      { path: "*", element: <Navigate to={Paths.STATUS_404} /> },
    ],
  },
];

export default routes;
```

## Setup common components

- Dựng các components dùng chung cho form dự án.

- Trong folder `components`:

  - Tạo components `Checkbox`.
  - Tạo components `Input`.
  - Tạo components `Label`.

- Trong các components `Checkbox`, `Input`, `Label` lần lượt tạo 2 file `index.tsx` và `styled.ts`.

```jsx
// Checkbox/index.tsx
import { Checkbox as CheckboxMui, CheckboxProps } from "@mui/material";
import React from "react";
import { BPCheckedIconStyled, BPIconStyled } from "./styled";

const Checkbox: React.FC<CheckboxProps> = (props) => {
  return (
    <CheckboxMui
      disableRipple
      color={props.color ? props.color : "default"}
      checkedIcon={
        <BPCheckedIconStyled
          sx={{
            backgroundColor: props.color
              ? `${props.color}.main`
              : "primary.main",
          }}
        />
      }
      icon={<BPIconStyled />}
      {...props}
    />
  );
};

export default Checkbox;
```

```jsx
// Checkbox/styled.ts
import { styled } from "@mui/material";

export const BPIconStyled = styled("span")(({ theme }) => ({
  borderRadius: 3,
  width: 19,
  height: 19,
  marginLeft: "4px",
  boxShadow:
    theme.palette.mode === "dark"
      ? `0 0 0 1px ${theme.palette.grey[200]}`
      : `inset 0 0 0 1px ${theme.palette.grey[300]}`,
  backgroundColor: "transparent",

  ".Mui-focusVisible &": {
    outline:
      theme.palette.mode === "dark"
        ? `0px auto ${theme.palette.grey[200]}`
        : `0px auto  ${theme.palette.grey[300]}`,
    outlineOffset: 2,
  },
  "input:hover ~ &": {
    backgroundColor:
      theme.palette.mode === "dark"
        ? theme.palette.primary
        : theme.palette.primary,
  },
  "input:disabled ~ &": {
    boxShadow: "none",
    background: theme.palette.grey[100],
  },
}));

export const BPCheckedIconStyled = styled(BPIconStyled)({
  boxShadow: "none",
  width: 19,
  height: 19,
  "&:before": {
    display: "block",
    width: 19,
    height: 19,
    backgroundImage:
      "url(\"data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3E%3Cpath" +
      " fill-rule='evenodd' clip-rule='evenodd' d='M12 5c-.28 0-.53.11-.71.29L7 9.59l-2.29-2.3a1.003 " +
      "1.003 0 00-1.42 1.42l3 3c.18.18.43.29.71.29s.53-.11.71-.29l5-5A1.003 1.003 0 0012 5z' fill='%23fff'/%3E%3C/svg%3E\")",
    content: '""',
  },
});
```

- Component Input.

```jsx
// Input/index.tsx
import { FormHelperText, TextFieldProps } from "@mui/material";
import React, { ReactNode } from "react";
import { TextFieldStyled } from "./styled";
import Label from "../Label";

type Props = TextFieldProps & {
  label?: string,
  errorMessage?: string,
  renderInput?: (props: TextFieldProps) => ReactNode,
};

const Input: React.FC<Props> = ({
  label,
  errorMessage,
  renderInput,
  ...props
}) => {
  return (
    <React.Fragment>
      {label && <Label>{label}</Label>}
      {renderInput ? renderInput(props) : <TextFieldStyled {...props} />}
      {errorMessage && (
        <FormHelperText error id="standard-weight-helper-text-email-login">
          {errorMessage}
        </FormHelperText>
      )}
    </React.Fragment>
  );
};

export default Input;
```

```jsx
// Input/styled.ts
import { TextField, styled } from "@mui/material";

export const TextFieldStyled = styled(TextField)(({ theme }) => ({
  "& .MuiOutlinedInput-input::-webkit-input-placeholder": {
    color: theme.palette.text.secondary,
    opacity: "0.8",
  },
  "& .MuiOutlinedInput-input.Mui-disabled::-webkit-input-placeholder": {
    color: theme.palette.text.secondary,
    opacity: "1",
  },
  "& .Mui-disabled .MuiOutlinedInput-notchedOutline": {
    borderColor: theme.palette.grey[200],
  },
}));
```

- Component Label.

```jsx
// Label/index.tsx
import { TypographyProps } from "@mui/material";
import { LabelStyled } from "./styled";
import React from "react";

const Label: React.FC<TypographyProps> = (props) => {
  return (
    <LabelStyled
      variant="subtitle1"
      fontWeight={600}
      component="label"
      {...props}
    />
  );
};

export default Label;
```

```jsx
// Label/styled.ts
import { Typography, styled } from "@mui/material";

export const LabelStyled = styled(Typography)({
  marginBottom: "5px",
  marginTop: "25px",
  display: "block",
  fontWeight: "600",
});
```

## Setup UI Login/Register page

- Trong folder `types`:

  - Tạo file `auth.ts` định nghĩa type `ILoginFormData`,`ILoginResponse`, `IToken` cho login page.

- Trong file `en.json` và `vi.json` thêm các text cần translation của login/register page.

- Trong folder `pages` tạo folder `Authentication`.
- Trong `Authentication` tiếp tục tạo `Login` page và `Register` page.

- Trong page `Login`:

  - Dựng UI theo design.
  - Thêm translation text.
  - Tạo function `_onLogin` cơ bản.

- Trong page `Register`:

  - Dựng UI theo design.
  - Thêm translation text.
  - Tạo function `_onRegister` cơ bản.

```jsx
// types/auth.ts
export interface ILoginFormData {
  email: string;
  password: string;
}

export interface IToken {
  id: string;
  accessToken: string;
  refreshToken: string;
  countRefreshToken: number;
}

export interface ILoginResponse {
  id: string;
  name: string;
  email: string;
  active: boolean;
  token: string;
  role: any;
  profileImage: string;
  createdUser: string;
  updatedUser: string;
  createdAt: string;
  updatedAt: string;
}
```

- File en.json và vi.json.

```jsx
// languages/en.json
{
  ...
 "MESSAGE_ERROR": {
        "requiredEmail": "Please typing email!",
        "requiredPassword": "Please typing password!"
    },
    "AUTH": {
        "welcomeToCFD": "Welcome to CFD",
        "yourAdminDashboard": "Your Admin Dashboard",
        "signIn": "Sign in",
        "forgotPassword?": "Forgot Password ?",
        "email": "Email",
        "password": "Password",
        "rememberThisDevice": "Remeber this device"
    }
  ...
}
```

```jsx
// languages/vi.json
{
  ...
  "MESSAGE_ERROR": {
        "requiredEmail": "Vui lòng nhập email!",
        "requiredPassword": "Vui lòng nhập password!"
    },
    "AUTH": {
        "welcomeToCFD": "Chào mừng đến với CFD",
        "yourAdminDashboard": "Your Admin Dashboard",
        "signIn": "Đăng nhập",
        "forgotPassword?": "Quên mật khẩu ?",
        "email": "Email",
        "password": "Mật khẩu",
        "rememberThisDevice": "Ghi nhớ đăng nhập"
    }
  ...
}
```

- Login page.

```jsx
// Authentication/Login/index.tsx
import Checkbox from "@/components/Checkbox";
import Input from "@/components/Input";
import { ILoginFormData } from "@/types/auth";
import {
  Box,
  Button,
  FormControlLabel,
  FormGroup,
  Stack,
  Typography,
} from "@mui/material";
import React from "react";
import { useForm } from "react-hook-form";
import { useTranslation } from "react-i18next";
import { Link } from "react-router-dom";

const Login = () => {
  const { t } = useTranslation();
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm < ILoginFormData > {};

  const _onLogin = async (data: ILoginFormData) => {
    //...
  };

  return (
    <React.Fragment>
      <Typography fontWeight="700" variant="h3" mb={1}>
        {t("AUTH.welcomeToCFD")}
      </Typography>

      {t("AUTH.yourAdminDashboard")}

      <form onSubmit={handleSubmit(_onLogin)}>
        <Input
          label={t("AUTH.email")}
          placeholder={t("AUTH.email")}
          fullWidth
          inputProps={{
            ...register("email", {
              required: t("MESSAGE_ERROR.requiredEmail"),
            }),
          }}
          errorMessage={errors?.email?.message}
        />
        <Input
          placeholder={t("AUTH.password")}
          fullWidth
          label={t("AUTH.password")}
          type="password"
          inputProps={{
            ...register("password", {
              required: t("MESSAGE_ERROR.requiredPassword"),
            }),
          }}
          errorMessage={errors?.password?.message}
        />
        <Stack
          justifyContent="space-between"
          direction="row"
          alignItems="center"
          my={2}
        >
          <FormGroup>
            <FormControlLabel
              control={<Checkbox defaultChecked />}
              label={t("AUTH.rememberThisDevice")}
            />
          </FormGroup>
          <Typography
            component={Link}
            to="/auth/forgot-password"
            fontWeight="500"
            sx={{
              textDecoration: "none",
              color: "primary.main",
            }}
          >
            {t("AUTH.forgotPassword?")}
          </Typography>
        </Stack>
        <Box>
          <Button
            color="primary"
            variant="contained"
            size="large"
            fullWidth
            type="submit"
          >
            {t("AUTH.signIn")}
          </Button>
        </Box>
      </form>
    </React.Fragment>
  );
};

export default Login;
```

- Register page.

```jsx
// Authentication/Register/index.tsx
import Checkbox from "@/components/Checkbox";
import Input from "@/components/Input";
import { authActions } from "@/store/auth/AuthSlice";
import { ILoginFormData } from "@/types/auth";
import {
  Box,
  Button,
  FormControlLabel,
  FormGroup,
  Stack,
  Typography,
} from "@mui/material";
import React from "react";
import { useForm } from "react-hook-form";
import { useTranslation } from "react-i18next";
import { Link } from "react-router-dom";

const Register = () => {
  const { t } = useTranslation();
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm < ILoginFormData > {};

  const _onRegister = async (data: ILoginFormData) => {};

  return (
    <React.Fragment>
      <Typography fontWeight="700" variant="h3" mb={1}>
        {t("AUTH.welcomeToCFD")}
      </Typography>

      {t("AUTH.yourAdminDashboard")}

      <form onSubmit={handleSubmit(_onLogin)}>
        <Input
          label={t("AUTH.email")}
          placeholder={t("AUTH.email")}
          fullWidth
          inputProps={{
            ...register("email", {
              required: t("MESSAGE_ERROR.requiredEmail"),
            }),
          }}
          errorMessage={errors?.email?.message}
        />
        <Input
          placeholder={t("AUTH.password")}
          fullWidth
          label={t("AUTH.password")}
          type="password"
          inputProps={{
            ...register("password", {
              required: t("MESSAGE_ERROR.requiredPassword"),
            }),
          }}
          errorMessage={errors?.password?.message}
        />
        <Stack
          justifyContent="space-between"
          direction="row"
          alignItems="center"
          my={2}
        >
          <FormGroup>
            <FormControlLabel
              control={<Checkbox defaultChecked />}
              label={t("AUTH.rememberThisDevice")}
            />
          </FormGroup>
          <Typography
            component={Link}
            to="/auth/forgot-password"
            fontWeight="500"
            sx={{
              textDecoration: "none",
              color: "primary.main",
            }}
          >
            {t("AUTH.forgotPassword?")}
          </Typography>
        </Stack>
        <Box>
          <Button
            color="primary"
            variant="contained"
            size="large"
            fullWidth
            type="submit"
          >
            Register
          </Button>
        </Box>
      </form>
    </React.Fragment>
  );
};

export default Register;
```

- Trong file `App.tsx`:
  - Gọi hàm `initLanguage`mỗi khi App chạy để init translation.

```jsx
...

import { initLanguage } from "./languages";
import { useEffect } from "react";

...

function App() {
  ...

  useEffect(() => {
    initLanguage();
  }, []);

  return (
   ...
  );
}

export default App;

```

## Setup axios, auth service and auth redux

### 1. Setup axiosInstance cơ bản

- Chạy lệnh `yarn add axios js-cookie` để cài đặt `axios` và `js-cookie` (thư viện làm việc với cookie).

- Trong `src` tạo file `.env` định nghĩa biến môi trường cho dự án.
- Trong folder `constants`:

  - Tạo file `environments.ts` và định nghĩa `ENV`,`BASE_URL`.
  - Trong file `storage.ts` thêm `token` key và `refresthTokenCount` key

- Trong folder `utils`:
  - Tạo file `token.ts` bao gồm các method dùng CRUD token.
  - Tạo file `axiosInstance.ts` và setup axios dùng call api cho dự án.

```jsx
// src/.env
VITE_ENV = "development";
VITE_BASE_URL = "https://cmscourse-api.cfdcircle.vn/api/v1/admin";
```

```jsx
// constants/environments.ts
// ref: https://vitejs.dev/guide/env-and-mode.html
const ENV = import.meta.env.VITE_ENV || "development";
const BASE_URL = import.meta.env.VITE_BASE_URL;

export { ENV, BASE_URL };
```

```jsx
// constants/storage.ts
export const STORAGE = {
  token: "cfd_token",
  language: "cfd_language",
  refreshTokenCount: "cfd_refreshtoken_count",
};
```

- Trong file `token.ts`.

```jsx
// utils/token.ts
import { STORAGE } from '@/constants/storage';
import { IToken } from '@/types/auth';
import Cookies from 'js-cookie';

export const localToken = {
    get: () => JSON.parse(localStorage.getItem(STORAGE.token) as string),
    set: (token: any) => localStorage.setItem(STORAGE.token, JSON.stringify(token)),
    remove: () => localStorage.removeItem(STORAGE.token),
};

export const cookieToken = {
    get: (): IToken =>
        JSON.parse(
            (Cookies.get(STORAGE.token) !== undefined
                ? Cookies.get(STORAGE.token)
                : null) as string,
        ),
    set: (token: unknown) => Cookies.set(STORAGE.token, JSON.stringify(token)),
    remove: () => Cookies.remove(STORAGE.token),
};

const tokenMethod = {
    get: () => {
        return cookieToken.get();
    },
    set: (token: IToken) => {
        cookieToken.set(token);
    },
    remove: () => {
        cookieToken.remove();
    },
};

export default tokenMethod;
  7 changes: 6 additions & 1 deletion7
yarn.lock
```

- Trong file `axiosInstance.ts`.

```jsx
// utils/axiosInstance.ts
import axios from "axios";
import tokenMethod from "./token";
import { BASE_URL } from "@/constants/environments";
import { authService } from "@/services/auth";

const axiosInstance = axios.create({
  baseURL: BASE_URL,
});

axiosInstance.interceptors.response.use(
  (response) => {
    return response?.data;
  },
  async (error) => {
    return error.config;
  }
);

axiosInstance.interceptors.request.use(
  (config) => {
    config.headers.Authorization = `Bearer ${tokenMethod.get()?.accessToken}`;
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

export default axiosInstance;
```

### 2. Setup auth service

- Trong folder `types`:

  - Tạo file `role.ts` định nghĩa type `IRole`.
  - Tạo file `user.ts`định nghĩa type `IUser`.

- Trong folder `services`:

  - Tạo file `auth.ts`.
  - Trong file `auth.ts` định nghĩa `authService`gồm các method `login`,`refresh`,`getProfile`

- Trong file `axiosInstance.ts` handle refresh token hệ thống.

```jsx
// types/user.ts
export interface IUser {
  id: string;
  name: string;
  email: string;
  active: boolean;
  token: string;
  role: string;
  profileImage: string;
  createdUser: CreateUpdateUser;
  updatedUser: CreateUpdateUser;
  createdAt: string;
  updatedAt: string;
}

export interface CreateUpdateUser extends Pick<IUser, "id" | "email"> {}
```

- Trong file `role.ts`.

```jsx
// types/role.ts
import { CreateUpdateUser } from "./user";

export interface IRole {
  id: string;
  name: string;
  isAdmin: boolean;
  permissions: string[];
  active: boolean;
  createdUser: CreateUpdateUser;
  updatedUser: CreateUpdateUser;
  createdAt: string;
  updatedAt: string;
}
```

- Trong file `services/auth.ts`.

```jsx
// services/auth.ts
import { IUser } from '../types/user';
import { ILoginResponse } from '@/types/auth';
import { ILoginFormData } from '@/types/auth';
import axiosInstance from '@/utils/axiosInstance';

const basePath = '/user' as const;

export const authService = {
    login(payload: ILoginFormData): Promise<{ data: ILoginResponse }> {
        return axiosInstance.post(`${basePath}/login`, payload);
    },
    refresh(refreshToken: string): Promise<{ data: ILoginResponse }> {
        return axiosInstance.put(`${basePath}/refresh`, {
            refreshToken,
        });
    },
    getProfile(id: string): Promise<IUser> {
        return axiosInstance.get(`${basePath}/${id}`);
    },
};
```

- Trong `axiosInstance`.

```jsx
// utils/axiosInstance.ts

...

axiosInstance.interceptors.response.use(
    (response) => {
        return response?.data;
    },
    async (error) => {
        const originalRequest = error.config;

        let countRefreshToken = Number(tokenMethod.get()?.countRefreshToken) || 0;
        if (
            (error.response?.status === 403 || error.response?.status === 401) &&
            !!!originalRequest._retry &&
            countRefreshToken <= 3 &&
            tokenMethod.get()
        ) {
            originalRequest._retry = true;
            try {
                const res = await authService.refresh(tokenMethod.get()?.refreshToken);
                countRefreshToken += 1;

                const { id, token } = res?.data || {};
                tokenMethod.set({
                    id: id,
                    accessToken: token,
                    refreshToken: token,
                    countRefreshToken: countRefreshToken,
                });
                originalRequest.headers.Authorization = `Bearer ${token}`;

                return axiosInstance(originalRequest);
            } catch (error) {
                tokenMethod.remove();
            }
        }
        return Promise.reject(error);
    },
);

...

export default axiosInstance;
```

### 3. Setup authen redux

- Trong folder `store` tạo folder `auth` handle cho module.
- Trong folder `auth` tạo file `AuthSlice.ts`.
- Trong `AuthSlice.ts`:

  - Khởi tạo `initialState` và `AuthSlice` với `createSlice`.
  - Dùng `createAsyncThunk` khởi tạo method `handleLogin` và `handleGetProfile`.
  - Export `authActions` và `AuthSlice.reducer`.

- Trong file `store/index.ts`:
  - Khai báo auth reducer bằng `AuthReducer` từ `auth` module.

```jsx
// auth/AuthSlice.ts
import { authService } from "@/services/auth";
import { ILoginFormData } from "@/types/auth";
import tokenMethod from "@/utils/token";
import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";

interface StateType {}

const initialState: StateType = {};

export const AuthSlice = createSlice({
  name: "auth",
  initialState,
  reducers: {
    reset: (state: StateType) => {
      state = { ...initialState };
    },
  },
});

export const handleLogin = createAsyncThunk(
  "auth/login",
  async (
    args: {
      payload: ILoginFormData,
      onSuccess?: VoidFunction,
      onFailed?: VoidFunction,
    },
    thunkApi
  ) => {
    const { payload, onSuccess, onFailed } = args;
    try {
      const res = await authService.login(payload);
      const { id, token } = res?.data || {};
      thunkApi.dispatch(handleGetProfile(id));
      tokenMethod.set({
        id: id,
        accessToken: token,
        refreshToken: token,
        countRefreshToken: 0,
      });
      onSuccess?.();
      return res?.data;
    } catch (error) {
      onFailed?.();
      return thunkApi.rejectWithValue(error);
    }
  }
);

export const handleGetProfile = createAsyncThunk(
  "auth/getProfile",
  async (id: string, thunkApi) => {
    try {
      const profileRes = await authService.getProfile(id);
      return profileRes;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);

export const authActions = {
  ...AuthSlice.actions,
  handleLogin,
  handleGetProfile,
};

export default AuthSlice.reducer;
```

```jsx
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { useDispatch, useSelector } from 'react-redux';
import CustomizerReducer from './customizer/CustomizerSlice';
import AuthReducer from './auth/AuthSlice';

export const store = configureStore({
  reducer: {
    customizer: CustomizerReducer,
  },
    reducer: {
        customizer: CustomizerReducer,
        auth: AuthReducer,
    },
});

// https://redux-toolkit.js.org/tutorials/typescript#define-typed-hooks
// Infer the `AppState` and `AppDispatch` types from the store itself
export type AppState = ReturnType<typeof store.getState>;
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch;

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<AppState>();

export default store;

```

- Trong `Login` page xử lý:
  - Dùng `dispatch` đẩy `authActions.handleLogin` trong auth slice.
    - Nếu thành công `navigate` về Dashboard.
    - Nếu thất bại `alert` thông báo.

```jsx
// Login/index.tsx
...
import { Paths } from '@/constants/paths';
import { useAppDispatch } from '@/store';
import { authActions } from '@/store/auth/AuthSlice';
import {  useNavigate } from 'react-router-dom';
...

const Login = () => {
    const dispatch = useAppDispatch();
    const navigate = useNavigate();

  ...

    const _onLogin = async (data: ILoginFormData) => {
        return await dispatch(
            authActions.handleLogin({
                payload: data,
                onSuccess: () => navigate(Paths.ROOT),
                onFailed: () => alert('Login failed!'),
            }),
        );
    };

    return (
    ...
    );
};

export default Login;
```

- Trong `MenuLayout`:
  - Tạo `useEffect` kiểm tra gọi `tokenMethod.get()?.id`.
  - Nếu có data `dispatch` actions ``authActions.handleGetProfile` để gọi lấy thông tin user về và lưu trữ trong redux.

```jsx
// MenuLayout/index.tsx
...
import {  useAppDispatch } from '@/store';
import { authActions } from '@/store/auth/AuthSlice';
import tokenMethod from '@/utils/token';
...

const MenuLayout: FC = () => {
  ...

  const dispatch = useAppDispatch();

  ...

  useEffect(() => {
    if (tokenMethod.get()?.id) {
      dispatch(authActions.handleGetProfile(tokenMethod.get().id));
    }
  }, []);

    return (
      <>...</>
    )

export default MenuLayout.
```

- Trong component `ProfileHeader` thực hiện xử lý method `_onLogout`.
- Dùng `tokenMethod.remove` để xóa token khỏi cookie.
- Dùng `authActions.reset` để reset state trong auth slice.
- Navigate về `Login` page.

```jsx
...

import { Link, useNavigate } from "react-router-dom";
import tokenMethod from "@/utils/token";
import { Paths } from "@/constants/paths";
import { useAppDispatch } from "@/store";
import { authActions } from "@/store/auth/AuthSlice";

const ProfileHeader = () => {
...

  const navigate = useNavigate();
  const dispatch = useAppDispatch();

...

  const _onLogout = () => {
    tokenMethod.remove();
    dispatch(authActions.reset());
    navigate(Paths.AUTHENTICATION);
  };

  return (
  ...
  );
};

export default ProfileHeader;

```
