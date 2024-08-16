# Dashboard page

- Dựng dashboard page.
- Cây trúc thư mục.

```jsx
src
...
|_ pages
    |_ Dashboard
        |_ components
            |_DashboardCard.tsx
            |_RevenueGrowthLine.tsx
        |_index.tsx
        |_LastestContacts.tsx
        |_LastestOrder.tsx
        |_RevenueUpdates.tsx
        |_TopCards.tsx
        |_YearlyBreakup.tsx
        |_YearlyEarnings.tsx
...
```

## Chia components

- Chia bố cục dạng grid.

```jsx
import Breadcrumb from "@/components/Breadcrumb";
import { Box, Grid } from "@mui/material";
import React, { useEffect } from "react";
import { useTranslation } from "react-i18next";
import TopCards from "./TopCards";
import RevenueUpdates from "./RevenueUpdates";
import YearlyBreakup from "./YearlyBreakup";
import YearlyEarnings from "./YearlyEarnings";
import LatestOrder from "./LastestOrder";
import LatestContacts from "./LastestContacts";
import { useAppDispatch } from "@/store";
import PageContainerLayout from "@/layouts/PageContainerLayout";

const DashboardPage: React.FC = () => {
  const { t } = useTranslation();
  return (
    <PageContainerLayout
      title={t("DASHBOARD.cfdDashboard")}
      description={t("DASHBOARD.thisIsDashboardPage")}
    >
      <Box>
        <Breadcrumb title="Dashboard" items={breadCrumbs} />
        <Grid container spacing={3}>
          <Grid item xs={12} lg={12}>
            <TopCards />
          </Grid>
          <Grid item xs={12} lg={8} display="flex">
            <RevenueUpdates />
          </Grid>
          <Grid item xs={12} lg={4}>
            <Grid container spacing={3}>
              <Grid item xs={12} sm={6} lg={12}>
                <YearlyBreakup />
              </Grid>
              <Grid item xs={12} sm={6} lg={12}>
                <YearlyEarnings />
              </Grid>
            </Grid>
          </Grid>
          <Grid item xs={12} lg={12}>
            <LatestOrder />
          </Grid>
          <Grid item xs={12} lg={12}>
            <LatestContacts />
          </Grid>
        </Grid>
      </Box>
    </PageContainerLayout>
  );
};

export default DashboardPage;
```

- Dùng lệnh `yarn add react-helmet` để cài đặt package.
- Trong folder `layouts` tạo `PageContainerLayout` để gắn `title`,`meta` cho page.
- Trong `PageContainerLayout` tạo file `index.tsx`.

```jsx
// PageContainerLayout/index.tsx
import { Helmet } from "react-helmet";

type Props = {
  description?: string,
  children: JSX.Element | JSX.Element[],
  title?: string,
};

const PageContainerLayout = ({ title, description, children }: Props) => (
  <div>
    <Helmet>
      <title>{title}</title>
      <meta name="description" content={description} />
    </Helmet>
    {children}
  </div>
);

export default PageContainerLayout;
```

## Xử lý logic

### 1. Chuẩn bị data và common component

- Trong folder `store` thêm `dashboard` module.
- Trong `dashboard` tạo `DashboardSlice.ts`.
- Trong `dashboard/DashboardSlice.ts`:
  - Định nghĩa `StateType` và `initialState`.
  - Khởi tạo `DashboardSlice`.
  - Thêm các async thunk function `handleGetMemberStatistic`, `handleGetRevenueStatistic`, `handleGetLastOrders`và `handleGetLastContacts` để get thông tin cần thiết.
  - Trong `extraReducers` bắt trạng thái các function thêm lưu data vào store.
  - Trong `store/index.ts` cấu hình thêm `dashboardReducer` vào `configureStore`.
  - Dùng function `shouldUpdate` trong `utils/validation` để kiểm tra xem cần thiết update lại data trong store hay không.
  - Dùng function `totalRevenueYear` trong `utils/calculator` để tính revenue.
  - Export `dashboardActions` bao gồm các actions mặc định và async thunk actions.

```jsx
// store/dashboard/DashboardSlice.ts
import { contactService } from "@/services/contact";
import { dashboardService } from "@/services/dashboard";
import { orderService } from "@/services/order";
import { IContact } from "@/types/contact";
import { IMemberStatistic, IRevenueStatistic } from "@/types/dashboard";
import { IOrder, IOrderQuery } from "@/types/order";
import { totalRevenueYear } from "@/utils/calculator";
import { shouldUpdate } from "@/utils/validation";
import { createAsyncThunk, createSlice, current } from "@reduxjs/toolkit";

interface StateType {
  memberStatistic?: IMemberStatistic;
  memberStatisticLoading: boolean;
  revenueStatistic?: IRevenueStatistic;
  totalRevenueData?: { thisYear: number, prevYear: number };
  revenueStatisticLoading?: boolean;
  lastOrders?: IOrder[] | [];
  lastOrderLoading?: boolean;
  lastContacts?: IContact[] | [];
  lastContactLoading?: boolean;
}

const initialState: StateType = {
  memberStatisticLoading: false,
  revenueStatisticLoading: false,
  lastOrders: [],
  lastOrderLoading: false,
  lastContacts: [],
  lastContactLoading: false,
};

const { actions, reducer: dashboardReducer } = createSlice({
  name: "dashboard",
  initialState,
  reducers: {
    reset: (state: StateType) => {
      state = { ...initialState };
    },
  },
  extraReducers: (builder) => {
    builder
      // ---- Get member statistic ---- //
      .addCase(handleGetMemberStatistic.fulfilled, (state, action) => {
        const getState = current(state);
        if (shouldUpdate(getState?.memberStatistic, action.payload)) {
          state.memberStatistic = action.payload;
        }
        state.memberStatisticLoading = false;
      })
      .addCase(handleGetMemberStatistic.pending, (state) => {
        state.memberStatisticLoading = true;
      })
      .addCase(handleGetMemberStatistic.rejected, (state) => {
        state.memberStatisticLoading = false;
      })
      // ---- Get revenue statistic ---- //
      .addCase(handleGetRevenueStatistic.fulfilled, (state, action) => {
        const getState = current(state);
        if (shouldUpdate(getState?.revenueStatistic, action.payload)) {
          state.revenueStatistic = action.payload;
          state.totalRevenueData = {
            thisYear: totalRevenueYear(action.payload.thisYear),
            prevYear: totalRevenueYear(action.payload.prevYear),
          };
        }
        state.revenueStatisticLoading = false;
      })
      .addCase(handleGetRevenueStatistic.pending, (state) => {
        state.revenueStatisticLoading = true;
      })
      .addCase(handleGetRevenueStatistic.rejected, (state) => {
        state.revenueStatisticLoading = false;
      })
      // ---- Get last orders ---- //
      .addCase(handleGetLastOrders.fulfilled, (state, action) => {
        const getState = current(state);
        if (shouldUpdate(getState?.lastOrders, action.payload)) {
          state.lastOrders = action.payload;
        }
        state.lastOrderLoading = false;
      })
      .addCase(handleGetLastOrders.pending, (state) => {
        state.lastOrderLoading = true;
      })
      .addCase(handleGetLastOrders.rejected, (state) => {
        state.lastOrderLoading = false;
      })
      // ---- Get last contacts ---- //
      .addCase(handleGetLastContacts.fulfilled, (state, action) => {
        const getState = current(state);
        if (shouldUpdate(getState?.lastContacts, action.payload)) {
          state.lastContacts = action.payload;
        }
        state.lastContactLoading = false;
      })
      .addCase(handleGetLastContacts.pending, (state) => {
        state.lastContactLoading = true;
      })
      .addCase(handleGetLastContacts.rejected, (state) => {
        state.lastContactLoading = false;
      });
  },
});

export const handleGetMemberStatistic = createAsyncThunk(
  "dashboard/memberStatistic",
  async (
    args: {
      fromDate?: string,
      toDate?: string,
    },
    thunkApi
  ) => {
    try {
      const res = await dashboardService.getMemberStatistic(args);
      return res?.data;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);

export const handleGetRevenueStatistic = createAsyncThunk(
  "dashboard/revenueStatistic",
  async (
    args: {
      year?: number,
    },
    thunkApi
  ) => {
    try {
      const res = await dashboardService.getRevenuesStatistic(args);
      return res?.data;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);

export const handleGetLastOrders = createAsyncThunk(
  "dashboard/orders",
  async (args: Partial<IOrderQuery>, thunkApi) => {
    try {
      const res = await orderService.getOrders(args);
      return res?.data?.orders;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);
export const handleGetLastContacts = createAsyncThunk(
  "dashboard/contacts",
  async (args: Partial<IOrderQuery>, thunkApi) => {
    try {
      const res = await contactService.getContacts(args);
      return res?.data?.subscribes;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);

const dashboardActions = {
  actions,
  handleGetMemberStatistic,
  handleGetRevenueStatistic,
  handleGetLastOrders,
  handleGetLastContacts,
};
export { dashboardActions, dashboardReducer };
```

```jsx
// store/index.ts

...

import { dashboardReducer } from "./dashboard/DashboardSlice";

export const store = configureStore({
  reducer: {
    customizer: CustomizerReducer,
    instructor: instructorReducer,
    dashboard: dashboardReducer,
  },
});

...

export default store;

```

- Trong folder `types`:
  - Tạo file `dashboard.ts` để định nghĩa type `IMemberStatistic`,`IRevenueStatistic`.
  - Tạo file `order.ts` để định nghĩa type `IOrder`,`IOrderQuery`.
  - Tạo file `contact.ts` để định nghĩa type `IContact`.

```jsx
// types/dashboard.ts
export interface IMemberStatistic {
  users: number;
  students: number;
  instructors: number;
  subscribes: number;
}

export interface IRevenueStatistic {
  thisYear: { [month: string]: number }[];
  prevYear: { [month: string]: number }[];
}
```

```jsx
// types/order.ts
import { IQuery } from "./general";
import { CreateUpdateUser } from "./user";

export interface IOrderQuery extends IQuery {}

export interface IOrder {
  id: string;
  name: string;
  slug: string;
  phone: string;
  course: Course;
  customer: Customer;
  type: string;
  paymentMethod: string;
  createdUser: CreateUpdateUser;
  updatedUser: CreateUpdateUser;
  createdAt: string;
  updatedAt: string;
}

export interface Course {
  name: string;
  id: string;
}

export interface Customer {
  firstName: string;
  lastName: string;
  email: string;
  id: string;
}
```

```jsx
// types/contact.ts
import { IQuery } from "./general";
import { CreateUpdateUser } from "./user";

export interface IContactQuery extends IQuery {}

export interface IContact {
  id: string;
  name: string;
  slug: string;
  title: string;
  description: string;
  email: string;
  phone: string;
  image: string;
  active: boolean;
  sortOrder: number;
  createdUser: CreateUpdateUser;
  updatedUser: CreateUpdateUser;
  createdAt: string;
  updatedAt: string;
}
```

- Trong folder `services`:
  - Tạo file `dashboard.ts` để định nghĩa `dashboardService`.
  - Tạo file `order.ts` để định nghĩa `orderService`.
  - Tạo file `contact.ts` để định nghĩa `contactService`.

```jsx
// services/dashboard.ts
import { IMemberStatistic, IRevenueStatistic } from '@/types/dashboard';
import axiosInstance from '@/utils/axiosInstance';

const basePath = '/dashboard' as const;

export const dashboardService = {
    getMemberStatistic(payload: {
        fromDate?: string;
        toDate?: string;
    }): Promise<{ data: IMemberStatistic }> {
        return axiosInstance.get(`${basePath}`, { params: payload });
    },
    getRevenuesStatistic(payload: { year?: number }): Promise<{ data: IRevenueStatistic }> {
        return axiosInstance.get(`${basePath}/order-split-by-months`, { params: payload });
    },
};
```

```jsx
// services/order.ts
import { PaginationType } from '@/types/general';
import { IOrder, IOrderQuery } from '@/types/order';
import axiosInstance from '@/utils/axiosInstance';

const basePath = '/orders' as const;

type TGetOrderResponse = { orders: IOrder[]; pagination: PaginationType };

export const orderService = {
    getOrders(payload: Partial<IOrderQuery>): Promise<{ data: TGetOrderResponse }> {
        return axiosInstance.get(`${basePath}`, { params: payload });
    },
};
```

```jsx
// services/contact.ts
import { IContact, IContactQuery } from '@/types/contact';
import { PaginationType } from '@/types/general';
import axiosInstance from '@/utils/axiosInstance';

const basePath = '/contacts' as const;

type TGetContactResponse = { subscribes: IContact[]; pagination: PaginationType };

export const contactService = {
    getContacts(payload: Partial<IContactQuery>): Promise<{ data: TGetContactResponse }> {
        return axiosInstance.get(`${basePath}`, { params: payload });
    },
};
```

- Trong `utils/validation`.

```jsx
// utils/validation.ts
const shouldUpdate = (oldValue?: string | object | any[], newValue?: string | object | any[]) => {
    if (!oldValue) return true;
    if (typeof oldValue === 'string' && typeof newValue === 'string') return oldValue !== newValue;
    return JSON.stringify(oldValue)?.length !== JSON.stringify(newValue)?.length;
};

...

export { shouldUpdate };
```

- Trong `utils/calculator.ts`.

```jsx
// utils/calculator.ts
import { IRevenueStatistic } from "@/types/dashboard";

const totalRevenueYear = (data?: IRevenueStatistic["thisYear"]) => {
  if (!data) return 0;
  return data?.reduce((acc, curr) => acc + +Object.values(curr)?.[0], 0);
};

export { totalRevenueYear };
```

- Trong folder `src/components` tạo comonent `AlertNotification`,`Select`.
- Trong `AlertNotification` tạo file `index.tsx` và `styled.ts`.
- Trong `AlertNotification/index.tsx`:
  - Định nghĩa type `IAlert`.
  - Khởi tạo các ref `refAlerts`, `refDispatch` bên ngoài component.
  - Khởi tạo `alerts` state để quản lý list alert.
  - Dùng `refDispatchCurrent` lưu trữ method `setAlerts` khi component mounted.
  - Khởi tạo `AlertNotification.useAlertNotification`.
  - Trong `AlertNotification.useAlertNotification`:
    - Tạo `cancelInteral` quản lý trạng thái cancel internal.
    - Tạo state `onAlertActions`chứa các actions `setAlert`,`warning`,`info`,`success` và `close`.
    - Dùng `useEffect` handle auto clear alert.
  - Trong `App.tsx` thêm component `AlertNotification`.
- Trong component `Select` tạo file `index.tsx`.
- Trong component `Loading` thêm props `isFullSize`.

```jsx
// AlertNotification/index.tsx
import React, { useEffect, useState } from 'react';
import { MainStyled, AlertStyled } from './styled';

export interface IAlert {
    type: 'error' | 'warning' | 'info' | 'success';
    content: string;
}

const refAlerts = React.createRef<IAlert[]>();
const refDispatch = React.createRef();
let refAlertsCurrent = refAlerts.current as IAlert[];
let refDispatchCurrent = refDispatch.current as React.Dispatch<React.SetStateAction<IAlert[]>>;
refAlertsCurrent = [];

const AlertNotification = () => {
    const [alerts, setAlerts] = useState<IAlert[]>(refAlertsCurrent || []);

    useEffect(() => {
        refDispatchCurrent = setAlerts;
    }, []);

    if (!!!alerts.length) return null;

    return (
        <MainStyled>
            {alerts?.map((alert, index) => {
                return (
                    <AlertStyled
                        severity={alert.type}
                        key={index}
                        onClose={() => setAlerts(alerts.filter((_, idx) => idx !== index))}
                    >
                        {alert.content}
                    </AlertStyled>
                );
            })}
        </MainStyled>
    );
};

AlertNotification.useAlertNotification = ({
    autoCloseTime = 2000,
}: { autoCloseTime?: number } | undefined = {}) => {
    const [cancelInteral, setCancelInterval] = useState(false);
    const [onAlertActions] = useState({
        setAlert: (data: IAlert) => {
            if (refAlertsCurrent?.length === 5) onAlertActions.close();
            refDispatchCurrent?.([...(refAlertsCurrent || []), data]);
            refAlertsCurrent?.push(data);
            setCancelInterval(false);
        },
        warning: (content: string) => onAlertActions.setAlert({ type: 'warning', content }),
        info: (content: string) => onAlertActions.setAlert({ type: 'info', content }),
        success: (content: string) => onAlertActions.setAlert({ type: 'success', content }),
        error: (content: string) => onAlertActions.setAlert({ type: 'error', content }),
        close: () => {
            const alerts = [...refAlertsCurrent];
            alerts?.shift();
            refDispatchCurrent?.(alerts);
            refAlertsCurrent = alerts;
        },
    });

    useEffect(() => {
        if (!autoCloseTime || cancelInteral) return;
        const timer = setInterval(() => {
            onAlertActions.close();
            if (!!!refAlertsCurrent?.length) setCancelInterval(true);
        }, autoCloseTime);

        return () => clearTimeout(timer);
    }, [autoCloseTime, cancelInteral]);

    return [onAlertActions];
};
export default AlertNotification;
```

```jsx
// AlertNotification/styled.ts
import { Alert, Box, styled } from "@mui/material";

const MainStyled = styled(Box)({
  display: "flex",
  flexDirection: "column",
  gap: "6px 0",
  position: "fixed",
  zIndex: 1000,
  bottom: 80,
  right: 10,
  maxWidth: "300px",
});

const AlertStyled = styled(Alert)({});

export { MainStyled, AlertStyled };
```

```jsx
import AlertNotification from "./components/AlertNotification";
...

function App() {

  ...

  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <LocalizationProvider dateAdapter={AdapterDayjs}>
        <ScrollToTop>{routing}</ScrollToTop>
      </LocalizationProvider>
       <AlertNotification />
    </ThemeProvider>
  );
}

export default App;

```

- Trong `Select/index.tsx`.

```jsx
// components/Select/index.tsx
import { Select as SelectMui, SelectProps } from "@mui/material";
import React from "react";

type Props = SelectProps & {};

const Select: React.FC<Props> = ({ ...props }) => <SelectMui {...props} />;

export default Select;
```

```jsx
// components/Loading/index.tsx
import { Box, CircularProgress, SxProps } from "@mui/material";
import React, { ReactNode } from "react";
import { LoadingOverlayStyled } from "./styled";

interface LoadingProps {
  isLoading?: boolean;
  children: ReactNode;
  styles?: SxProps;
  isFullSize?: boolean;
  className?: string;
  isLoadingForPage?: boolean;
}

const Loading: React.FC<LoadingProps> = ({
  children,
  isLoading = false,
  isLoadingForPage = false,
  styles,
  isFullSize,
  className,
}) => {
  const loadingPageStyles: SxProps = isLoadingForPage
    ? {
        width: "100vw",
        height: "100vh",
        position: "fixed",
      }
    : {};

  return (
    <Box
      sx={{
        width: isFullSize ? "100%" : "fit-content",
        height: isFullSize ? "100%" : "fit-content",
        position: "relative",
        ...styles,
      }}
      component={"div"}
      className={className}
    >
      {isLoading && (
        <Box
          sx={{
            width: "100%",
            height: "100%",
            position: "absolute",
            top: 0,
            left: 0,
            zIndex: 2,
            ...loadingPageStyles,
          }}
        >
          <LoadingOverlayStyled>
            <CircularProgress size={32} thickness={5} />
          </LoadingOverlayStyled>
        </Box>
      )}
      {children}
    </Box>
  );
};

export default Loading;
```

### 2. Handle Dashboard page.

- Thêm `breadCrumbs` page.
- Dùng function `initChartData` gọi dispatch các async thunk actions trong `dashboardActions` để get data cần thiết.
- Trong `useEffect` gọi `initChartData`, `onAlertActions.info` lần đầu và `setInterval` sau mỗi 5 phút gọi lại `initChartData`, `onAlertActions.info`.

```jsx
// Dashboard/index.tsx
import Breadcrumb from "@/components/Breadcrumb";
import { Box, Grid } from "@mui/material";
import React, { useEffect } from "react";
import { useTranslation } from "react-i18next";
import TopCards from "./TopCards";
import RevenueUpdates from "./RevenueUpdates";
import YearlyBreakup from "./YearlyBreakup";
import YearlyEarnings from "./YearlyEarnings";
import LatestOrder from "./LastestOrder";
import LatestContacts from "./LastestContacts";
import { useAppDispatch } from "@/store";
import { dashboardActions } from "@/store/dashboard/DashboardSlice";
import { useSearchQuery } from "@/hooks";
import dayjs from "dayjs";
import PageContainerLayout from "@/layouts/PageContainerLayout";
import AlertNotification from "@/components/AlertNotification";

const DashboardPage: React.FC = () => {
  const [onAlertActions] = AlertNotification.useAlertNotification({
    autoCloseTime: 5000,
  });
  const { t } = useTranslation();
  const dispatch = useAppDispatch();
  const currentYear = dayjs().year();
  const { searchParams } = useSearchQuery();
  const breadCrumbs = [
    {
      to: "/",
      title: "Home",
    },
    {
      title: t("dashboard"),
    },
  ];

  const initChartData = () => {
    dispatch(dashboardActions.handleGetMemberStatistic({}));
    dispatch(
      dashboardActions.handleGetRevenueStatistic({
        year: Number(searchParams?.year) || currentYear,
      })
    );
    dispatch(dashboardActions.handleGetLastOrders({ page: 1, limit: 6 }));
    dispatch(dashboardActions.handleGetLastContacts({ page: 1, limit: 6 }));
  };

  useEffect(() => {
    initChartData();
    onAlertActions.info(t("DASHBOARD.dashboardIUpdateAfter5minutes!"));
    const recallTime = 60000 * 5; // 5 minutes
    const timer = setInterval(() => {
      initChartData();
      onAlertActions.info(t("DASHBOARD.dashboardIUpdateAfter5minutes!"));
    }, recallTime);
    return () => clearInterval(timer);
  }, []);

  return (
    <PageContainerLayout
      title={t("DASHBOARD.cfdDashboard")}
      description={t("DASHBOARD.thisIsDashboardPage")}
    >
      <Box>
        <Breadcrumb title="Dashboard" items={breadCrumbs} />
        <Grid container spacing={3}>
          {/* column */}
          <Grid item xs={12} lg={12}>
            <TopCards />
          </Grid>
          {/* column */}
          <Grid item xs={12} lg={8} display="flex">
            <RevenueUpdates />
          </Grid>
          {/* column */}
          <Grid item xs={12} lg={4}>
            <Grid container spacing={3}>
              <Grid item xs={12} sm={6} lg={12}>
                <YearlyBreakup />
              </Grid>
              <Grid item xs={12} sm={6} lg={12}>
                <YearlyEarnings />
              </Grid>
            </Grid>
          </Grid>
          {/* column */}
          <Grid item xs={12} lg={12}>
            <LatestOrder />
          </Grid>
          {/* column */}
          <Grid item xs={12} lg={12}>
            <LatestContacts />
          </Grid>
        </Grid>
      </Box>
    </PageContainerLayout>
  );
};

export default DashboardPage;
```
