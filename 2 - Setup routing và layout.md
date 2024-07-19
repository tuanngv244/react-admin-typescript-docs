# Setup routing và layout

React Router DOM là một thư viện phổ biến được sử dụng để quản lý việc điều hướng và định tuyến trang trong ứng dụng React. Nó cung cấp một API mạnh mẽ và linh hoạt để tạo ra các trải nghiệm điều hướng mượt mà và đáp ứng.

## Setup routing

- Sử dụng lệnh `yarn add react-router-dom` để cài đặt vào dự án.

### 1. Khởi tạo cấu trúc pages cơ bản:

- Trong folder `pages`:
- Tạo folder `Dashboard` cho module dashboard.
- Trong folder `Dashboard` tạo file `index.tsx`.

```jsx
// Dashboard/index.tsx
import React from "react";

const DashboardPage: React.FC = () => {
  return <div>DashboardPage</div>;
};

export default DashboardPage;
```

- Tạo folder `Course` cho module course.
- Trong folder `Course` tạo thêm các folder `Create`, `Detail` và `List` cho các page con của module course.
- Trong folder `Create`, `Detail` và `List` lần lượt tạo file `index.tsx`.

```jsx
// List/index.tsx
import React from "react";

const CourseListPage: React.FC = () => {
  return <div>CourseListPage</div>;
};

export default CourseListPage;
```

```jsx
// Detail/index.tsx
import React from "react";

const CourseDetailPage: React.FC = () => {
  return <div>CourseDetailPage</div>;
};

export default CourseDetailPage;
```

```jsx
// Create/index.tsx
import React from "react";

const CourseCreatePage: React.FC = () => {
  return <div>CourseCreatePage</div>;
};

export default CourseCreatePage;
```

- Tạo folder `Authentication` cho module authentication.
- Trong folder `Authentication` tạo file `index.tsx`.

```jsx
import React from "react";

const AuthenticationPage: React.FC = () => {
  return <div>AuthenticationPage</div>;
};

export default AuthenticationPage;
```

- Tạo folder `Status404` cho các route không tồn tại.
- Trong folder `Status404` tạo file `index.tsx`.

```jsx
import React from "react";

const Status404: React.FC = () => {
  return <div>Status404</div>;
};

export default Status404;
```

### 2. Cấu hình routes:

- Trong folder `constants` tạo file `paths.ts`.
- Định nghĩa enum Paths routing cho dự án.

```jsx
/*--- PATH ENUM ---*/
// Define the base paths
const COURSES_BASE = '/course';

// Define path enum
export enum Paths {
    ROOT = '/',

    AUTHENTICATION = '/authen',

    DASHBOARD = '/',

    COURSES = COURSES_BASE,
    CREATE_COURSE = `${COURSES_BASE}/create`,
    COURSE_DETAIL = `${COURSES_BASE}/:id`,

    LOGIN = "/authentication/login",
    REGISTER = "/authentication/register",

    STATUS_404 = '/404',
}
```

- Trong folder `configs`:
  - Tạo file `routes.tsx`để định nghĩa routing.
  - Dùng `lazy` của React để import page.
  - Cấu hình routing cơ bản theo dạng [Route Array](https://reactrouter.com/en/main/hooks/use-routes#useroutes).
  - Import path cho các module đã được định nghĩa enum trong `constants/paths`.
- Trong file `App.tsx`:
  - Dùng `useRoutes` để khởi tạo render ra cây route.
- Trong folder `components`:
  - Tạo component `Spninner` dùng cho các trường hợp loading.
- Trong file `main.tsx`:
- Dùng `BrowserRouter` bao ngoài `App` để sử dụng context route.
- Dùng `Suspense` bao ngoài cùng với fallback là `Spinner`.

```jsx
import { Paths } from "@/constants/paths";
import { Navigate, RouteObject } from "react-router-dom";
import { lazy } from "react";
// Authentication
const Authentication = lazy(() => import("../pages/Authentication"));
// Dashboard
const Dashboard = lazy(() => import("../pages/Dashboard"));
// Course
const CourseListPage = lazy(() => import("../pages/Course/List"));
const CourseCreatePage = lazy(() => import("../pages/Course/Create"));
const CourseDetailPage = lazy(() => import("../pages/Course/Detail"));
// ...
// Status
const Status404 = lazy(() => import("../pages/Status404"));

/*--- ROUTE ARRAY ---*/
const routes: RouteObject[] = [
  {
    path: Paths.ROOT,
    element: <></>,
    children: [
      {
        path: "",
        element: <Dashboard />,
      },
      {
        path: Paths.COURSES,
        children: [
          {
            path: "",
            element: <CourseListPage />,
          },
          {
            path: Paths.CREATE_COURSE,
            element: <CourseCreatePage />,
          },
          {
            path: Paths.COURSE_DETAIL,
            element: <CourseDetailPage />,
          },
        ],
      },
      // ...
    ],
  },
  {
    path: Paths.ROOT,
    element: <></>,
    children: [
      { path: Paths.AUTHENTICATION, element: <Authentication /> },
      { path: Paths.STATUS_404, element: <Status404 /> },
      { path: "*", element: <Navigate to={Paths.STATUS_404} /> },
    ],
  },
];

export default routes;
```

```jsx
//App.tsx
import { useTheme } from "./theme";
import { CssBaseline, ThemeProvider } from "@mui/material";
import { useRoutes } from "react-router-dom";
import routes from "./configs/routes";

function App() {
  const theme = useTheme();
  const routing = useRoutes(routes);

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {routing}
    </ThemeProvider>
  );
}

export default App;
```

```jsx
// components/Spinner/index.tsx
import React from "react";
import "./spinner.css";

const Spinner: React.FC = () => (
  <div className="fallback-spinner">
    <div className="loading component-loader">
      <div className="effect-1 effects" />
      <div className="effect-2 effects" />
      <div className="effect-3 effects" />
    </div>
  </div>
);
export default Spinner;
```

```jsx
// components/Spinner/spinner.css
.fallback-spinner {
  position: relative;
  display: flex;
  height: 100vh;
  width: 100%;
}
.loading {
  position: absolute;
  left: calc(50% - 35px);
  top: 50%;
  width: 55px;
  height: 55px;
  border-radius: 50%;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  border: 3px solid transparent;
}
.effect-1,
.effect-2 {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 3px solid transparent;
  border-left: 3px solid #2962ff;
  border-radius: 50%;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

.effect-1 {
  animation: rotate 1s ease infinite;
}
.effect-2 {
  animation: rotateOpacity 1s ease infinite 0.1s;
}
.effect-3 {
  width: 100%;
  height: 100%;
  border: 3px solid transparent;
  border-left: 3px solid #2962ff;
  -webkit-animation: rotateOpacity 1s ease infinite 0.2s;
  animation: rotateOpacity 1s ease infinite 0.2s;
  border-radius: 50%;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

.loading .effects {
  transition: all 0.3s ease;
}
.fallback-logo {
  position: absolute;
  left: calc(50% - 45px);
  top: 40%;
}

@keyframes rotate {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(1turn);
    transform: rotate(1turn);
  }
}
@keyframes rotateOpacity {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
    opacity: 0.1;
  }
  100% {
    -webkit-transform: rotate(1turn);
    transform: rotate(1turn);
    opacity: 1;
  }
}
```

```jsx
// main.tsx
import { Provider } from 'react-redux';
import App from './App.tsx';
import store from './store/index.tsx';
import { BrowserRouter } from 'react-router-dom';
import { Suspense } from 'react';
import Spinner from './components/Spinner/index.tsx';
import ReactDOM from "react-dom/client";

ReactDOM.createRoot(document.getElementById('root')!).render(
    <Suspense fallback={<Spinner />}>
        <Provider store={store}>
            <BrowserRouter>
                <App />
            </BrowserRouter>
        </Provider>
    </Suspense>,
);
```

- Thêm supense loader cho route.
- Trong folder `components`:
  - Tạo component `SupenseLoader`.
- Tạo folder `hocs`.
- Trong folder `hocs`:
  - Tạo file `withSuspenseLoader.tsx` dùng để custom load cho các page route.
- Trong file `routes.tsx`:
  - Bao `withSuspenseLoader` bao lại các page import.

```jsx
// components/SuspenseLoader/index.tsx
import React, { useEffect } from "react";
import NProgress from "nprogress";
import { Box, CircularProgress } from "@mui/material";

const SuspenseLoader: React.FC = () => {
  useEffect(() => {
    NProgress.start();

    return () => {
      NProgress.done();
    };
  }, []);
  return (
    <Box
      sx={{
        position: "fixed",
        left: 0,
        top: 0,
        width: "100%",
        height: "100%",
      }}
      display="flex"
      alignItems="center"
      justifyContent="center"
    >
      <CircularProgress size={64} disableShrink thickness={3} />
    </Box>
  );
};

export default SuspenseLoader;
```

```jsx
// hocs/withSuspenseLoader.tsx
import SuspenseLoader from '@/components/SuspenseLoader';
import { Suspense } from 'react';

/**
 * Loader
 * This HOC add Suspense - fallback to handle lazy loading of input Component
 */
const withSuspenseLoader =
  <P extends object>(Component: React.ComponentType<P>): React.FC<P> =>
  (props: P) => {
    return (
      <Suspense fallback={<SuspenseLoader />}>
        <Component {...props} />
      </Suspense>
    );
  };

export default withSuspenseLoader;
```

```jsx
import withSuspenseLoader from '@/hocs/withSuspenseLoader';

...

// Authentication
const Authentication = withSuspenseLoader(lazy(() => import('../pages/Authentication')));
// Dashboard
const Dashboard = withSuspenseLoader(lazy(() => import('../pages/Dashboard')));
// Course
const CourseListPage = withSuspenseLoader(lazy(() => import('../pages/Course/List')));
const CourseCreatePage = withSuspenseLoader(lazy(() => import('../pages/Course/Create')));
const CourseDetailPage = withSuspenseLoader(lazy(() => import('../pages/Course/Detail')));
// Status
const Status404 = withSuspenseLoader(lazy(() => import('../pages/Status404')));

...

export default routes;
```

- Trong folder `components` tạo component `ScrollToTop` để bắt mỗi khi redirect giữa các route tự động scroll lên đầu page.
- Trong file `App.tsx`:
  - Dùng `ScrollToTop` bao ngoài routing.

```jsx
// components/ScrollToTop.tsx
import { useEffect, ReactElement } from "react";
import { useLocation } from "react-router-dom";

interface ScrollToTopProps {
  children: ReactElement | null;
}

export default function ScrollToTop({
  children,
}: ScrollToTopProps): ReactElement | null {
  const { pathname } = useLocation();

  useEffect(() => {
    window.scrollTo({
      top: 0,
      left: 0,
      behavior: "smooth",
    });
  }, [pathname]);

  return children || null;
}
```

```jsx
//App.tsx
import { useTheme } from "./theme";
import { CssBaseline, ThemeProvider } from "@mui/material";
import { useRoutes } from "react-router-dom";
import routes from "./configs/routes";
import ScrollToTop from "./components/ScrollToTop";

function App() {
  const theme = useTheme();
  const routing = useRoutes(routes);

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <ScrollToTop>{routing}</ScrollToTop>
    </ThemeProvider>
  );
}

export default App;
```

## Setup Layout

- Trong folder `components`:

  - Tạo component `PrivateRoute/index.tsx`.

- Trong folder `layouts`:
  - Tạo `BlankLayout/index.tsx`.
  - Tạo `MenuLayout/index.tsx`.

```jsx
// components/PrivateRoute/index.tsx
import { Paths } from "@/constants/paths";
import React from "react";
import { Navigate } from "react-router-dom";

interface PrivateRouteProps {
  redirectTo: string;
  element: React.ReactNode;
}

const PrivateRoute: React.FC<PrivateRouteProps> = ({
  redirectTo = Paths.AUTHENTICATION,
  element,
}) => {
  // TODO: handle get token for condition
  return true ? element : <Navigate to={redirectTo} replace />;
};

export default PrivateRoute;
```

```jsx
// layouts/BlankLayout/index.tsx
import React from "react";
import { Outlet } from "react-router-dom";

const BlankLayout: React.FC = () => {
  return (
    <div className="BlankLayout">
      <Outlet />
    </div>
  );
};

export default BlankLayout;
```

```jsx
// layouts/MenuLayout/index.tsx
import React from "react";
import { Outlet } from "react-router-dom";

const MenuLayout: React.FC = () => {
  return (
    <div className="MenuLayout">
      <Outlet />
    </div>
  );
};

export default MenuLayout;
```

- Trong file `configs/route.tsx`:
  - Dùng `PrivateRoute` làm element của root route.
  - Dùng `MenuLayout` làm element render trong PrivateRoute và có rediectPath `Authentication`.
  - Dùng `BlankLayout` làm layout cho các page `Authentication`, `Status404` và các route `*`(Không có route trong dự án).

```jsx
...

import PrivateRoute from '../components/PrivateRoute';

/*--- LAYOUTS IMPORT ---*/
import BlankLayout from '@/layouts/BlankLayout';
import MenuLayout from '@/layouts/MenuLayout';

...

const routes: RouteObject[] = [
    {
        path: Paths.ROOT,
        element: <PrivateRoute redirectTo={Paths.AUTHENTICATION} element={<MenuLayout />} />,
        children: [
            {
                path: '',
                element: <Dashboard />,
            },
            {
                path: Paths.COURSES,
                children: [
                    {
                        path: '',
                        element: <CourseListPage />,
                    },
                    {
                        path: Paths.CREATE_COURSE,
                        element: <CourseCreatePage />,
                    },
                    {
                        path: Paths.COURSE_DETAIL,
                        element: <CourseDetailPage />,
                    },
                ],
            },
            // ...
        ],
    },
    {
      path: Paths.ROOT,
      element: <BlankLayout />,
      children: [
          { path: Paths.AUTHENTICATION, element: <Authentication /> },
          { path: Paths.STATUS_404, element: <Status404 /> },
          { path: '*', element: <Navigate to={Paths.STATUS_404} /> },
      ],
  },
];

...

export default routes;
```
