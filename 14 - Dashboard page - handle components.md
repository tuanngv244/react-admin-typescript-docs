### 3. Handle TopCards

- Trong `dashboard` tạo component `TopCards.tsx`.
  - Trong `TopCards.tsx`
  - Trong folder `assets/icons` thêm các icon `ICUserMale`,`ICBriefCase`,`ICConnect` và `ICSpeechBubble`.
  - Dùng `useAppSelector` lấy data `memberStatistic`.
  - Tiếp tục lấy các data `users`,`students`,`subscribes` và `instructors` render giao diện.

```jsx
// Dashboard/TopCards.tsx
import { SVGS } from "@/assets/icons";
import { useAppSelector } from "@/store";
import { Box, CardContent, Grid, Typography } from "@mui/material";
import { useMemo } from "react";
import { useTranslation } from "react-i18next";

interface cardType {
  icon: Function;
  title: string;
  digits: string;
  bgcolor: string;
}

const TopCards = () => {
  const { t } = useTranslation();
  const memberStatistic = useAppSelector(
    (state) => state.dashboard.memberStatistic
  );

  const topcards: cardType[] = useMemo(
    () => [
      {
        icon: () => <SVGS.ICUserMale />,
        title: t("DASHBOARD.users"),
        digits: `${memberStatistic?.users || 0}`,
        bgcolor: "primary",
      },
      {
        icon: () => <SVGS.ICBriefCase />,
        title: t("DASHBOARD.students"),
        digits: `${memberStatistic?.students || 0}`,
        bgcolor: "warning",
      },
      {
        icon: () => <SVGS.ICConnect />,
        title: t("DASHBOARD.register"),
        digits: `${memberStatistic?.subscribes || 0}`,
        bgcolor: "secondary",
      },
      {
        icon: () => <SVGS.ICSpeechBubble />,
        title: t("DASHBOARD.instructors"),
        digits: `${memberStatistic?.instructors || 0}`,
        bgcolor: "success",
      },
    ],
    [memberStatistic]
  );

  return (
    <Grid container spacing={3} className="top-cards">
      {topcards.map((topcard, i) => (
        <Grid item xs={12} sm={3} lg={3} key={i}>
          <Box bgcolor={topcard.bgcolor + ".light"} textAlign="center">
            <CardContent>
              {topcard.icon()}
              <Typography
                color={topcard.bgcolor + ".main"}
                mt={1}
                variant="subtitle1"
                fontWeight={600}
              >
                {topcard.title}
              </Typography>
              <Typography
                color={topcard.bgcolor + ".main"}
                variant="h4"
                fontWeight={600}
              >
                {topcard.digits}
              </Typography>
            </CardContent>
          </Box>
        </Grid>
      ))}
    </Grid>
  );
};

export default TopCards;
```

### 4. Handle RevenueUpdates

- Dùng lệnh `yarn add react-apexcharts apexcharts` để cài đặt package charts.
- Trong `dashboard` tạo component `RevenueUpdates.tsx`.
- Trong `RevenueUpdates.tsx`:
  - Dùng `useAppSelector` lấy data `revenueData`,`totalRevenueData`.
  - Thêm function `handleChange` để gọi `dashboardActions.handleGetRevenueStatistic` lấy data revenue mới theo year.
  - Tạo `optionscolumnchart` cấu hình revenue chart UI giao diện.
  - Tạo `seriescolumnchart` để render columns chart.
  - Tạo `totalEarnings` tính tổng thu nhập.
- Trong folder `constants` tạo folder `pages`.
- Trong `pages` tạo file `dashboard.ts` chứa các constants của dashboard module.
- Trong `dashboard/components`:
  - Tạo component `DashboardCard.tsx`.
- Trong `constants/pages/dashboard.ts`:
  - Tạo `TOTAL_COST`,`yearOptions`.

```jsx
// Dashboard/RevenueUpdates.tsx
import {
  Avatar,
  Box,
  Button,
  Grid,
  MenuItem,
  SelectChangeEvent,
  Stack,
  Typography,
} from "@mui/material";
import { useTheme } from "@mui/material/styles";
import { useMemo } from "react";
import Chart, { Props } from "react-apexcharts";
import DashboardCard from "./components/DashboardCard";
import Select from "@/components/Select";
import { useAppDispatch, useAppSelector } from "@/store";
import { formatCurrency } from "@/utils/transform";
import { useTranslation } from "react-i18next";
import { dashboardActions } from "@/store/dashboard/DashboardSlice";
import { useSearchQuery } from "@/hooks";
import dayjs from "dayjs";
import { TOTAL_COST, yearOptions } from "@/constants/pages/dashboard";

const RevenueUpdates = () => {
  const { t } = useTranslation();
  const currentYear = dayjs().year();
  const dispatch = useAppDispatch();
  const { searchParams, _onSetSearchParams } = useSearchQuery();

  const revenueData = useAppSelector(
    (state) => state.dashboard.revenueStatistic
  );
  const totalRevenueData = useAppSelector(
    (state) => state.dashboard.totalRevenueData
  );
  // const revenueLoading = useAppSelector((state) => state.dashboard.revenueStatisticLoading);

  const handleChange = (event: SelectChangeEvent<any>) => {
    const value = event.target.value;
    _onSetSearchParams({ year: value });
    dispatch(dashboardActions.handleGetRevenueStatistic({ year: value }));
  };

  // chart color
  const theme = useTheme();
  const primary = theme.palette.primary.main;
  const secondary = theme.palette.secondary.main;

  // chart
  const optionscolumnchart: Props = {
    chart: {
      type: "bar",
      fontFamily: "'Plus Jakarta Sans', sans-serif;",
      foreColor: "#adb0bb",
      toolbar: {
        show: false,
      },
    },
    colors: [
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
      primary,
    ],
    plotOptions: {
      bar: {
        borderRadius: 4,
        columnWidth: "60%",
        distributed: true,
        endingShape: "rounded",
      },
    },
    dataLabels: {
      enabled: false,
    },
    legend: {
      show: false,
    },
    grid: {
      yaxis: {
        lines: {
          show: false,
        },
      },
    },
    xaxis: {
      categories: [
        ["Jan"],
        ["Feb"],
        ["Mar"],
        ["Apr"],
        ["May"],
        ["June"],
        ["July"],
        ["Aug"],
        ["Sept"],
        ["Oct"],
        ["Nov"],
        ["Dec"],
      ],
      axisBorder: {
        show: false,
      },
    },
    yaxis: {
      labels: {
        show: false,
      },
    },
    tooltip: {
      theme: theme.palette.mode === "dark" ? "dark" : "light",
    },
  };
  const seriescolumnchart = useMemo(() => {
    return [
      {
        name: "",
        data:
          revenueData?.thisYear?.map((item) => Object.values(item)?.[0]) || [],
      },
    ];
  }, [revenueData?.thisYear]);

  const totalEarnings = !totalRevenueData?.thisYear
    ? 0
    : Number(totalRevenueData?.thisYear) - TOTAL_COST;

  return (
    <DashboardCard
      title={t("DASHBOARD.overviewOfRevenue")}
      // subtitle="Overview of Profit"
      action={
        <Select
          labelId="month-dd"
          id="month-dd"
          size="small"
          value={searchParams?.year || currentYear}
          onChange={handleChange}
        >
          {yearOptions.map((option, index) => (
            <MenuItem key={index} value={option.value}>
              {" "}
              {option.label}
            </MenuItem>
          ))}
        </Select>
      }
    >
      <Grid container spacing={3}>
        {/* column */}
        <Grid item xs={12} sm={8}>
          <Box height="350px" paddingTop="150px">
            <Chart
              options={optionscolumnchart}
              series={seriescolumnchart}
              type="bar"
              height="200px"
            />
          </Box>
        </Grid>
        {/* column */}
        <Grid item xs={12} sm={4}>
          <Stack spacing={3} mt={3}>
            <Stack direction="row" spacing={2} alignItems="center">
              <Box>
                <Typography variant="h3" fontWeight="700">
                  ${formatCurrency(totalEarnings || 0)}
                </Typography>
                <Typography variant="subtitle2" color="textSecondary">
                  {t("DASHBOARD.totalEarnings")}
                </Typography>
              </Box>
            </Stack>
          </Stack>
          <Stack spacing={3} my={5}>
            <Stack direction="row" spacing={2}>
              <Avatar
                sx={{
                  width: 9,
                  mt: "0.5rem !important",
                  height: 9,
                  bgcolor: primary,
                  svg: { display: "none" },
                }}
              ></Avatar>
              <Box>
                <Typography variant="subtitle1" color="textSecondary">
                  {t("DASHBOARD.revenueThisYear")}
                </Typography>
                <Typography variant="h5">
                  ${formatCurrency(totalRevenueData?.thisYear || 0)}
                </Typography>
              </Box>
            </Stack>
            <Stack direction="row" spacing={2}>
              <Avatar
                sx={{
                  width: 9,
                  mt: "0.5rem !important",
                  height: 9,
                  bgcolor: secondary,
                  svg: { display: "none" },
                }}
              ></Avatar>
              <Box>
                <Typography variant="subtitle1" color="textSecondary">
                  {t("DASHBOARD.revenueLastYear")}
                </Typography>
                <Typography variant="h5">
                  ${formatCurrency(totalRevenueData?.prevYear || 0)}
                </Typography>
              </Box>
            </Stack>
          </Stack>
          <Button color="primary" variant="contained" fullWidth>
            {t("DASHBOARD.viewFullReport")}
          </Button>
        </Grid>
      </Grid>
    </DashboardCard>
  );
};

export default RevenueUpdates;
```

```jsx
// dashboard/components/DashboardCard.tsx
import { useTheme } from "@mui/material/styles";
import { Card, CardContent, Typography, Stack, Box } from "@mui/material";
import { useAppSelector } from "@/store";

type Props = {
  title?: string,
  subtitle?: string,
  action?: JSX.Element,
  footer?: JSX.Element,
  cardheading?: string | JSX.Element,
  headtitle?: string | JSX.Element,
  headsubtitle?: string | JSX.Element,
  children?: JSX.Element,
  middlecontent?: string | JSX.Element,
};

const DashboardCard = ({
  title,
  subtitle,
  children,
  action,
  footer,
  cardheading,
  headtitle,
  headsubtitle,
  middlecontent,
}: Props) => {
  const customizer = useAppSelector((state) => state.customizer);

  const theme = useTheme();
  const borderColor = theme.palette.divider;

  return (
    <Card
      sx={{
        padding: 0,
        border: !customizer.isCardShadow ? `1px solid ${borderColor}` : "none",
        overflow: "hidden",
        height: "100%",
      }}
      elevation={customizer.isCardShadow ? 9 : 0}
      variant={!customizer.isCardShadow ? "outlined" : undefined}
    >
      {cardheading ? (
        <CardContent>
          <Typography variant="h5">{headtitle}</Typography>
          <Typography variant="subtitle2" color="textSecondary">
            {headsubtitle}
          </Typography>
        </CardContent>
      ) : (
        <CardContent sx={{ p: "30px" }}>
          {title ? (
            <Stack
              direction="row"
              spacing={2}
              justifyContent="space-between"
              alignItems={"center"}
              mb={3}
            >
              <Box>
                {title ? <Typography variant="h5">{title}</Typography> : ""}

                {subtitle ? (
                  <Typography variant="subtitle2" color="textSecondary">
                    {subtitle}
                  </Typography>
                ) : (
                  ""
                )}
              </Box>
              {action}
            </Stack>
          ) : null}

          {children}
        </CardContent>
      )}

      {middlecontent}
      {footer}
    </Card>
  );
};

export default DashboardCard;
```

- Trong `constants/pages/dashboard.ts`.

```jsx
// constants/pages/dashboard.ts
import dayjs from "dayjs";

const TOTAL_COST = 500000000; // example total cost for one year.

const yearOptions = Array.from({ length: 5 }).map((_, index) => {
  const currentYear = dayjs().year();
  return {
    value: currentYear - index,
    label: currentYear - index,
  };
});

export { yearOptions, TOTAL_COST };
```

### 5. Handle YearlyBreakup

- Trong `dashboard` tạo component `YearlyBreakup.tsx`.
- Trong `YearlyBreakup.tsx`:
  - Dùng `useAppSelector` lấy data `revenueData`,`totalRevenueData`.
  - Tạo `optionscolumnchart` cấu hình chart UI.
  - Tạo `seriescolumnchart` cấu hình render columns chart.
  - Tạo `yearlyBreakupValue` tính thu nhập thực năm.
  - Tạo `percentRevenueGrowth` tính phần trăm tăng trưởng thực năm.
- Trong `dashboard/components`:
  - Tạo component `RevenueGrowthLine.tsx`.

```jsx
// dashboard/YearlyBreakup.tsx
import Chart from "react-apexcharts";
import { useTheme } from "@mui/material/styles";
import { Grid, Stack, Typography, Avatar } from "@mui/material";
import { Props } from "react-apexcharts";
import DashboardCard from "./components/DashboardCard";
import { useAppSelector } from "@/store";
import { useMemo } from "react";
import { formatCurrency } from "@/utils/transform";
import dayjs from "dayjs";
import { useTranslation } from "react-i18next";
import RevenueGrowthLine from "./components/RevenueGrowthLine";
import { totalRevenueYear } from "@/utils/calculator";

const YearlyBreakup = () => {
  const { t } = useTranslation();
  const currentYear = dayjs().year();
  const revenueData = useAppSelector(
    (state) => state.dashboard.revenueStatistic
  );
  const totalRevenueData = useAppSelector(
    (state) => state.dashboard.totalRevenueData
  );

  // chart color
  const theme = useTheme();
  const primary = theme.palette.primary.main;
  const primarylight = theme.palette.primary.light;

  // chart
  const optionscolumnchart: Props = {
    chart: {
      type: "donut",
      fontFamily: "'Plus Jakarta Sans', sans-serif;",
      foreColor: "#adb0bb",
      toolbar: {
        show: false,
      },
      height: 155,
    },
    colors: [primarylight, primary],
    plotOptions: {
      pie: {
        startAngle: 0,
        endAngle: 360,
        donut: {
          size: "75%",
          background: "transparent",
        },
      },
    },
    tooltip: {
      enabled: false,
    },
    stroke: {
      show: false,
    },
    dataLabels: {
      enabled: false,
    },
    legend: {
      show: false,
    },
    responsive: [
      {
        breakpoint: 991,
        options: {
          chart: {
            width: 120,
          },
        },
      },
    ],
  };
  const seriescolumnchart = [
    Number(totalRevenueData?.prevYear),
    Number(totalRevenueData?.thisYear),
  ];

  const yearlyBreakupValue = useMemo(
    () =>
      totalRevenueYear(revenueData?.thisYear) -
      totalRevenueYear(revenueData?.prevYear),
    [revenueData]
  );

  const percentRevenueGrowth = useMemo(() => {
    const revenueThisYear = Number(totalRevenueData?.thisYear);
    const revenuePrevYear = Number(totalRevenueData?.prevYear);
    if (!totalRevenueData?.thisYear) return 0;
    if (!totalRevenueData?.prevYear) return 100;
    return ((revenueThisYear - revenuePrevYear) / revenuePrevYear) * 100 || 0;
  }, [totalRevenueData]);

  return (
    <DashboardCard title={t("DASHBOARD.yearlyBreakup")}>
      <Grid container spacing={3}>
        {/* column */}
        <Grid item xs={7} sm={7}>
          <Typography variant="h3" fontWeight="700">
            {}${formatCurrency(yearlyBreakupValue || 0)}
          </Typography>
          <Stack direction="row" spacing={1} mt={1} alignItems="center">
            <RevenueGrowthLine data={percentRevenueGrowth} />
          </Stack>
          <Stack spacing={3} mt={5} direction="row">
            <Stack direction="row" spacing={1} alignItems="center">
              <Avatar
                sx={{
                  width: 9,
                  height: 9,
                  bgcolor: primarylight,
                  svg: { display: "none" },
                }}
              ></Avatar>
              <Typography variant="subtitle2" color="textSecondary">
                {currentYear - 1}
              </Typography>
            </Stack>
            <Stack direction="row" spacing={1} alignItems="center">
              <Avatar
                sx={{
                  width: 9,
                  height: 9,
                  bgcolor: primary,
                  svg: { display: "none" },
                }}
              ></Avatar>
              <Typography variant="subtitle2" color="textSecondary">
                {currentYear}
              </Typography>
            </Stack>
          </Stack>
        </Grid>
        {/* column */}
        <Grid item xs={5} sm={5}>
          <Chart
            options={optionscolumnchart}
            series={seriescolumnchart}
            type="donut"
            height="130px"
          />
        </Grid>
      </Grid>
    </DashboardCard>
  );
};

export default YearlyBreakup;
```

```jsx
// dashboard/components/RevenueGrowthLine.tsx
import { Avatar, Typography } from "@mui/material";
import { IconArrowDownRight, IconArrowUpLeft } from "@tabler/icons-react";
import React from "react";
import { useTheme } from "@mui/material/styles";
import { useTranslation } from "react-i18next";

type Props = {
  data: number,
};

const RevenueGrowthLine: React.FC<Props> = ({ data }) => {
  const { t } = useTranslation();
  const theme = useTheme();
  const successlight = theme.palette.success.light;
  const errorlight = theme.palette.error.light;

  return data > 0 ? (
    <>
      <Avatar sx={{ bgcolor: successlight, width: 27, height: 27 }}>
        <IconArrowUpLeft width={20} color="#39B69A" />
      </Avatar>
      <Typography variant="subtitle2" fontWeight="600">
        +{data.toFixed(1)}%
      </Typography>
      <Typography variant="subtitle2" color="textSecondary">
        {t("DASHBOARD.lastYear")}
      </Typography>
    </>
  ) : (
    <>
      <Avatar sx={{ bgcolor: errorlight, width: 27, height: 27 }}>
        <IconArrowDownRight width={20} color="#FA896B" />
      </Avatar>
      <Typography variant="subtitle2" fontWeight="600">
        {data.toFixed(1)}%
      </Typography>
      <Typography variant="subtitle2" color="textSecondary">
        {t("DASHBOARD.lastYear")}
      </Typography>
    </>
  );
};

export default RevenueGrowthLine;
```

### 6. Handle YearlyEarnings

- Trong `dashboard` tạo component `YearlyEarnings.tsx`.
- Trong `YearlyEarnings.tsx`:
  - Dùng `useAppSelector` lấy data `revenueData`,`totalRevenueData`.
  - Tạo `optionscolumnchart` cấu hình chart UI.
  - Tạo `seriescolumnchart` cấu hình render columns chart.
  - Tạo `totalEarnings` tính tổng thu nhập năm.
  - Tạo `percentRevenueGrowth` tính phần trăm tổng tăng trưởng năm.
- Trong `dashboard/components`:
  - Tạo component `DashboardCard.tsx`.
  - Tạo component `RevenueGrowthLine.tsx`.

```jsx
// dashboard/YearlyEarnings.tsx
import { Fab, Stack, Typography } from "@mui/material";
import { useTheme } from "@mui/material/styles";
import { IconCurrencyDollar } from "@tabler/icons-react";
import Chart from "react-apexcharts";

import { TOTAL_COST } from "@/constants/pages/dashboard";
import { useAppSelector } from "@/store";
import { formatCurrency } from "@/utils/transform";
import { useMemo } from "react";
import { Props } from "react-apexcharts";
import { useTranslation } from "react-i18next";
import DashboardCard from "./components/DashboardCard";
import RevenueGrowthLine from "./components/RevenueGrowthLine";

const YearlyEarnings = () => {
  const { t } = useTranslation();
  const revenueData = useAppSelector(
    (state) => state.dashboard.revenueStatistic
  );
  const totalRevenueData = useAppSelector(
    (state) => state.dashboard.totalRevenueData
  );

  // chart color
  const theme = useTheme();
  const secondary = theme.palette.secondary.main;
  const secondarylight = theme.palette.secondary.light;

  // chart
  const optionscolumnchart: Props = {
    chart: {
      type: "area",
      fontFamily: "'Plus Jakarta Sans', sans-serif;",
      foreColor: "#adb0bb",
      toolbar: {
        show: false,
      },
      height: 60,
      sparkline: {
        enabled: true,
      },
      group: "sparklines",
    },
    stroke: {
      curve: "smooth",
      width: 2,
    },
    fill: {
      colors: [secondarylight],
      type: "solid",
      opacity: 0.05,
    },
    markers: {
      size: 0,
    },
    tooltip: {
      theme: theme.palette.mode === "dark" ? "dark" : "light",
      x: {
        show: false,
      },
    },
  };

  const seriescolumnchart = useMemo(() => {
    return [
      {
        name: "",
        color: secondary,
        data:
          revenueData?.thisYear?.map((item) => Object.values(item)?.[0]) || [],
      },
    ];
  }, [revenueData?.thisYear]);

  const totalEarnings = !totalRevenueData?.thisYear
    ? 0
    : Number(totalRevenueData?.thisYear) - TOTAL_COST;

  const percentRevenueGrowth = useMemo(() => {
    const earningThisYear = totalEarnings;
    const earningPrevYear = Number(totalRevenueData?.prevYear) - TOTAL_COST;

    if (!totalRevenueData?.thisYear) return 0;
    if (!totalRevenueData?.prevYear) return 100;
    return ((earningThisYear - earningPrevYear) / earningPrevYear) * 100 || 0;
  }, [totalRevenueData]);

  return (
    <DashboardCard
      title={t("DASHBOARD.yearlyEarnings")}
      action={
        <Fab color="secondary" size="medium">
          <IconCurrencyDollar width={24} />
        </Fab>
      }
      footer={
        <Chart
          options={optionscolumnchart}
          series={seriescolumnchart}
          type="area"
          height="60px"
        />
      }
    >
      <>
        <Typography variant="h3" fontWeight="700" mt="-20px">
          ${formatCurrency(totalEarnings)}
        </Typography>
        <Stack direction="row" spacing={1} my={1} alignItems="center">
          <RevenueGrowthLine data={percentRevenueGrowth} />
        </Stack>
      </>
    </DashboardCard>
  );
};

export default YearlyEarnings;
```

### 7. Handle LatestOrder

- Trong `dashboard` tạo component `LatestOrder.tsx`.
- Trong `LatestOrder.tsx`:

  - Dùng `useAppSelector` lấy data `orderData`,`orderLoading`.
  - Tạo table render UI data order.
  - Dùng `displayNullish` để check null data.

- Trong folder `utils/transform.ts` thêm function `displayNullish`.

```jsx
// dashboard/LatestOrder.tsx
import {
  Avatar,
  Box,
  Button,
  Chip,
  Stack,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Typography,
} from "@mui/material";
import DashboardCard from "./components/DashboardCard";
import { useAppSelector } from "@/store";
import dayjs from "dayjs";
import { DATE_FORMATS } from "@/constants/format";
import Loading from "@/components/Loading";
import { displayNullish } from "@/utils/transform";
import { useTranslation } from "react-i18next";

const LatestOrder = () => {
  const { t } = useTranslation();
  const orderData = useAppSelector((state) => state.dashboard.lastOrders);
  const orderLoading = useAppSelector(
    (state) => state.dashboard.lastOrderLoading
  );

  return (
    <Loading isFullSize isLoading={orderLoading}>
      <DashboardCard
        title="Latest orders"
        action={
          <Button color="primary" variant="contained">
            {t("DASHBOARD.viewAll")}
          </Button>
        }
      >
        <TableContainer>
          <Table
            aria-label="simple table"
            sx={{
              whiteSpace: "nowrap",
            }}
          >
            <TableHead>
              <TableRow>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.students")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.phone")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.courses")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.type")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.paymentMethod")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.date")}
                  </Typography>
                </TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {orderData?.map((basic) => (
                <TableRow key={basic.id}>
                  <TableCell>
                    <Stack direction="row" spacing={2}>
                      <Avatar sx={{ width: 40, height: 40 }}>
                        {basic?.name?.charAt(0)}
                      </Avatar>
                      <Box>
                        <Typography
                          color="textSecondary"
                          variant="subtitle2"
                          fontWeight={600}
                        >
                          {displayNullish(basic.name)}
                        </Typography>
                        <Typography
                          color="textSecondary"
                          fontSize="12px"
                          variant="subtitle2"
                        >
                          {displayNullish(basic.customer?.email)}
                        </Typography>
                      </Box>
                    </Stack>
                  </TableCell>
                  <TableCell>
                    <Typography
                      color="textSecondary"
                      variant="subtitle2"
                      fontWeight={400}
                    >
                      {displayNullish(basic.phone)}
                    </Typography>
                  </TableCell>
                  <TableCell>
                    <Typography
                      color="textSecondary"
                      variant="subtitle2"
                      fontWeight={600}
                    >
                      {displayNullish(basic.course?.name)}
                    </Typography>
                  </TableCell>
                  <TableCell>
                    <Chip
                      sx={{
                        bgcolor:
                          basic.type === "Online"
                            ? (theme) => theme.palette.primary.light
                            : basic.type === "Offline"
                            ? (theme) => theme.palette.warning.light
                            : (theme) => theme.palette.secondary.light,
                        color:
                          basic.type === "Online"
                            ? (theme) => theme.palette.primary.main
                            : basic.type === "Offline"
                            ? (theme) => theme.palette.warning.main
                            : (theme) => theme.palette.secondary.main,
                        borderRadius: "8px",
                      }}
                      size="small"
                      label={basic.type}
                    />
                  </TableCell>
                  <TableCell>
                    <Chip
                      sx={{
                        bgcolor:
                          basic.paymentMethod === "Transfer"
                            ? (theme) => theme.palette.secondary.light
                            : basic.paymentMethod === "Cash"
                            ? (theme) => theme.palette.success.light
                            : (theme) => theme.palette.secondary.light,
                        color:
                          basic.paymentMethod === "Transfer"
                            ? (theme) => theme.palette.secondary.main
                            : basic.paymentMethod === "Cash"
                            ? (theme) => theme.palette.success.main
                            : (theme) => theme.palette.secondary.main,
                        borderRadius: "8px",
                      }}
                      size="small"
                      label={basic.paymentMethod}
                    />
                  </TableCell>
                  <TableCell>
                    <Typography variant="subtitle2">
                      {dayjs(basic.createdAt).format(DATE_FORMATS.DATE)}
                    </Typography>
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
      </DashboardCard>
    </Loading>
  );
};

export default LatestOrder;
```

```jsx
// utils/transform.ts

const displayNullish = (data: string | number) => {
    return data ? data : '---';
};

...

export {displayNullish}
```

### 8. Hanlde LatestContacts

- Trong `dashboard` tạo component `LatestContacts.tsx`.
- Trong `LatestContacts.tsx`:

  - Dùng `useAppSelector` lấy data `contactData`,`contactLoading`.
  - Tạo table render UI data order.
  - Dùng `displayNullish` để check null data.

```jsx
// dashboard/LatestContacts.tsx
import {
  Avatar,
  Box,
  Button,
  Stack,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Typography,
} from "@mui/material";
import DashboardCard from "./components/DashboardCard";
import { useAppSelector } from "@/store";
import Loading from "@/components/Loading";
import dayjs from "dayjs";
import { DATE_FORMATS } from "@/constants/format";
import { displayNullish } from "@/utils/transform";
import { useTranslation } from "react-i18next";

const LatestContacts = () => {
  const { t } = useTranslation();
  const contactData = useAppSelector((state) => state.dashboard.lastContacts);
  const contactLoading = useAppSelector(
    (state) => state.dashboard.lastContactLoading
  );

  return (
    <Loading isLoading={contactLoading} isFullSize>
      <DashboardCard
        title="Latest contacts"
        action={
          <Button color="primary" variant="contained">
            {t("DASHBOARD.viewAll")}
          </Button>
        }
      >
        <TableContainer>
          <Table
            aria-label="simple table"
            sx={{
              whiteSpace: "nowrap",
            }}
          >
            <TableHead>
              <TableRow>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.users")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.subject")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.phone")}
                  </Typography>
                </TableCell>
                <TableCell>
                  <Typography variant="subtitle2" fontWeight={600}>
                    {t("DASHBOARD.date")}
                  </Typography>
                </TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {contactData?.map((basic) => (
                <TableRow key={basic.id}>
                  <TableCell>
                    <Stack direction="row" spacing={2}>
                      <Avatar sx={{ width: 40, height: 40 }}>
                        {basic?.name?.charAt(0)}
                      </Avatar>
                      <Box>
                        <Typography
                          color="textSecondary"
                          variant="subtitle2"
                          fontWeight={600}
                        >
                          {displayNullish(basic.name)}
                        </Typography>
                        <Typography
                          color="textSecondary"
                          fontSize="12px"
                          variant="subtitle2"
                        >
                          {displayNullish(basic.email)}
                        </Typography>
                      </Box>
                    </Stack>
                  </TableCell>
                  <TableCell>
                    <Typography
                      color="textSecondary"
                      variant="subtitle2"
                      fontWeight={600}
                    >
                      {displayNullish(basic.description)}
                    </Typography>
                  </TableCell>
                  <TableCell>
                    <Typography
                      color="textSecondary"
                      variant="subtitle2"
                      fontWeight={400}
                    >
                      {displayNullish(basic.phone)}
                    </Typography>
                  </TableCell>
                  <TableCell>
                    <Typography
                      color="textSecondary"
                      variant="subtitle2"
                      fontWeight={400}
                    >
                      {dayjs(basic.createdAt).format(DATE_FORMATS.DATE)}
                    </Typography>
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
      </DashboardCard>
    </Loading>
  );
};

export default LatestContacts;
```

### 9. Handle locales EN/VI

- Trong file `en.json` thêm translation for `DASHBOARD`.

```jsx
// locales/en.json
{
...
  "DASHBOARD": {
    "cfdDashboard": "CFD Dashboard",
    "thisIsDashboardPage": "This is Dashboard page",
    "totalEarnings": "Total Earnings",
    "revenueThisYear": "Revenue this year",
    "revenueLastYear": "Revenue last year",
    "overviewOfRevenue": "Overview of revenue",
    "viewAll": "View all",
    "viewFullReport": "View full report",
    "yearlyBreakup": "Yearly Breakup",
    "yearlyEarnings": "Yearly Earnings",
    "lastYear": "last year",
    "users": "Users",
    "subject": "Subject",
    "phone": "Phone",
    "date": "Date",
    "students": "Students",
    "courses": "Courses",
    "type": "Type",
    "paymentMethod": "Payment method",
    "register": "Register",
    "instructors": "Instructors",
    "dashboardIUpdateAfter5minutes!": "Dashboard information will be updated automatically every 5 minutes!"
  }
...
}
```

```jsx
// locales/vi.json
{
...
   "DASHBOARD": {
    "cfdDashboard": "CFD Dashboard",
    "thisIsDashboardPage": "This is Dashboard page",
    "totalEarnings": "Total Earnings",
    "revenueThisYear": "Revenue this year",
    "revenueLastYear": "Revenue last year",
    "overviewOfRevenue": "Overview of revenue",
    "viewAll": "View all",
    "viewFullReport": "View full report",
    "yearlyBreakup": "Yearly Breakup",
    "yearlyEarnings": "Yearly Earnings",
    "lastYear": "last year",
    "users": "Users",
    "subject": "Subject",
    "phone": "Phone",
    "date": "Date",
    "students": "Students",
    "courses": "Courses",
    "type": "Type",
    "paymentMethod": "Payment method",
    "register": "Register",
    "instructors": "Instructors",
    "dashboardIUpdateAfter5minutes!": "Dashboard information will be updated automatically every 5 minutes!"
  }
...
}
```
