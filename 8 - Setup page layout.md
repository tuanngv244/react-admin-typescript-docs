# Setup page layout

- Dựng layout chung cho các page trong dự án bao như list page, detail page ,create/update page ,...

## Setup list page layout

- Trong folder `assets` tạo folder `images`.
- Trong folder `images`:
  - Tải ảnh `img-chat-bc.png` từ design bỏ vào `images`.
  - Tạo file `index.ts`, tạo `IMAGES` khai báo images của dự án.

```jsx
// assets/images/index.ts
import ChatBc from "./img-chat-bc.png";

const IMAGES = {
  breadCrumb: {
    chatBc: ChatBc,
  },
  course: {
    // img here
  },
};

export default IMAGES;
```

- Trong folder `utils` tạo file `transform.ts` để modify các dữ liệu cần thiết.
- Trong `transform.ts` tạo function `toUpperCaseFirstLetter` để uppercase kí tự đầu.

```jsx
// utils/transform.ts
const toUpperCaseFirstLetter = (data: string) => {
  const result: string[] = [];
  data.split(" ").forEach((item) => {
    result.push(item[0].toUpperCase() + item.slice(1));
  });
  return result.join(" ");
};

export { toUpperCaseFirstLetter };
```

- Trong folder `components` tạo component `Breadcrumb/index.tsx`.
- Trong `Breadcrumb`:
  - Nhận props `items` để render list route.

```jsx
// components/Breadcrumb/index.tsx
import { Box, Breadcrumbs, Grid, Link, Theme, Typography } from "@mui/material";
import { NavLink } from "react-router-dom";

import { toUpperCaseFirstLetter } from "@/utils/transform";
import { IconCircle } from "@tabler/icons-react";
import { TypographyStyled } from "./styled";
import IMAGES from "@/assets/images";

interface BreadCrumbType {
  subtitle?: string;
  items?: { to?: string, title: string }[];
  title: string;
  children?: JSX.Element;
}

const Breadcrumb = ({ subtitle, items, title, children }: BreadCrumbType) => (
  <Grid
    container
    sx={{
      backgroundColor: "primary.light",
      borderRadius: (theme: Theme) => theme.shape.borderRadius / 4,
      p: "30px 25px 20px",
      marginBottom: "10px",
      position: "relative",
      overflow: "hidden",
    }}
  >
    <Grid item xs={12} sm={6} lg={8} mb={1}>
      <TypographyStyled variant="h5">{title}</TypographyStyled>
      <TypographyStyled
        color="textSecondary"
        variant="h6"
        fontWeight={400}
        mt={0.8}
        mb={0}
      >
        {subtitle}
      </TypographyStyled>
      <Breadcrumbs
        separator={
          <IconCircle
            size="5"
            fill="textSecondary"
            fillOpacity={"0.6"}
            style={{ margin: "0 5px" }}
          />
        }
        sx={{ alignItems: "center", mt: items ? "10px" : "" }}
        aria-label="breadcrumb"
      >
        {items
          ? items.map((item) => (
              <div key={item.title}>
                {item.to ? (
                  <Link
                    underline="none"
                    color="inherit"
                    component={NavLink}
                    to={item.to}
                  >
                    {toUpperCaseFirstLetter(item.title)}
                  </Link>
                ) : (
                  <Typography color="textPrimary">
                    {toUpperCaseFirstLetter(item.title)}
                  </Typography>
                )}
              </div>
            ))
          : ""}
      </Breadcrumbs>
    </Grid>
    <Grid item xs={12} sm={6} lg={4} display="flex" alignItems="flex-end">
      <Box
        sx={{
          display: { xs: "none", md: "block", lg: "flex" },
          alignItems: "center",
          justifyContent: "flex-end",
          width: "100%",
        }}
      >
        {children ? (
          <Box sx={{ top: "0px", position: "absolute" }}>{children}</Box>
        ) : (
          <>
            <Box sx={{ top: "0px", position: "absolute" }}>
              <img
                src={IMAGES.breadCrumb.chatBc}
                alt={IMAGES.breadCrumb.chatBc}
                width={"165px"}
              />
            </Box>
          </>
        )}
      </Box>
    </Grid>
  </Grid>
);

export default Breadcrumb;
```

- Trong `Breadcrumb` tạo file `styled.ts` để style cho breadcrumb.

```jsx
// Breadcrumb/styled.ts
import { Typography, styled } from "@mui/material";

export const TypographyStyled = styled(Typography)({
  textTransform: "capitalize",
  fontWeight: "600",
});
```

- Trong folder `layouts`tạo `ListPageLayout` dùng để bọc các list page.
- Trong `ListPageLayout/index.tsx`:
  - Nhận vào props `pageTitle`, `renderTable`, `createPath`, `hideSearch`,`searchPlaceholder`.
  - Dùng `renderTable` để render table của list page.
  - Dựng layout với `title` và `BoxHeadStyled`.
  - Title uppercase kí tự đầu tiên.
  - Trong `BoxHeadStyled` tạo Input và Button dùng cho page cần search và action tạo.
  - Tạo hàm `_onSearchChange` dùng cho các page search.
  - Dùng `_.debounce` bọc hàm `_onSearchChange`.

```jsx
// ListPageLayout/index.tsx
import Breadcrumb from "@/components/Breadcrumb";
import Input from "@/components/Input";
import { useSearchQuery } from "@/hooks/useSearchQuery";
import {
  OutlinedInputStyled,
  BoxHeadStyled,
} from "@/layouts/ListPageLayout/styled";
import { Box, Button, InputAdornment } from "@mui/material";
import { IconSearch } from "@tabler/icons-react";
import debounce from "lodash/debounce";
import React, { ChangeEvent, ReactNode } from "react";
import { useTranslation } from "react-i18next";
import { useNavigate } from "react-router-dom";

type ListPageLayoutProps = {
  pageTitle: string,
  renderTable: ReactNode,
  createPath?: string,
  hideSearch?: boolean,
  searchPlaceholder?: string,
  breadCrumbs: { title: string, to?: string }[],
};

const ListPageLayout: React.FC<ListPageLayoutProps> = ({
  pageTitle,
  renderTable,
  hideSearch,
  createPath,
  searchPlaceholder,
  breadCrumbs,
}) => {
  const { t } = useTranslation();
  const navigate = useNavigate();
  const { _onSetSearchParams } = useSearchQuery();
  const title = pageTitle.charAt(0).toUpperCase() + pageTitle.slice(1);

  const _onSearchChange = debounce((ev: ChangeEvent<HTMLInputElement>) => {
    _onSetSearchParams({ search: ev.target?.value });
  }, 500);

  return (
    <Box
      className="ListPageLayout"
      sx={{
        marginTop: "20px",
      }}
    >
      <Breadcrumb title={title} items={breadCrumbs || []} />
      <BoxHeadStyled>
        <Button
          onClick={() => navigate(createPath || location.pathname + "/create")}
          color="primary"
          variant="contained"
          size="large"
        >
          {t("COMMON.create")} {pageTitle}
        </Button>
        {!hideSearch && (
          <Input
            renderInput={() => (
              <OutlinedInputStyled
                onChange={_onSearchChange}
                placeholder={searchPlaceholder}
                className="customInput"
                startAdornment={
                  <InputAdornment position="start">
                    <IconSearch size={"16"} />
                  </InputAdornment>
                }
              />
            )}
          />
        )}
      </BoxHeadStyled>
      {renderTable}
    </Box>
  );
};

export default ListPageLayout;
```

- Trong `ListPageLayout` tạo `styled.ts` để style cho layout.

```jsx
// ListPageLayout/styled.ts
import { Box, OutlinedInput, styled } from "@mui/material";

const BoxRightHeadStyled = styled(Box)({
  display: "inline-flex",
  alignItems: "center",
  gap: "8px",
});

const BoxHeadStyled = styled(Box)({
  padding: "10px 0",
  display: "inline-flex",
  width: "100%",
  columnGap: "8px",
  alignItems: "center",
  justifyContent: "space-between",
  margin: "10px 0 ",
});

const OutlinedInputStyled = styled(OutlinedInput)({
  minWidth: "300px",
  height: "42.25px",
});

export { BoxRightHeadStyled, BoxHeadStyled, OutlinedInputStyled };
```

## Setup detail page layout

- Trong folder `components` tạo component `Loading` để loading cho page, layout, component,...
- Tong component `Loading`:
  - nhận props children để render content.
  - Dùng `isLoading` để control loading từ bên ngoài.

```jsx
// Loading/index.tsx
import { Box, CircularProgress, SxProps } from "@mui/material";
import React, { ReactNode } from "react";
import { LoadingOverlayStyled } from "./styled";

interface LoadingProps {
  isLoading: boolean;
  children: ReactNode;
  styles?: SxProps;
  isLoadingForPage?: boolean;
}

const Loading: React.FC<LoadingProps> = ({
  children,
  isLoading,
  isLoadingForPage = false,
  styles,
}) => {
  const loadingPageStyles: SxProps = isLoadingForPage
    ? {
        width: "100vw",
        height: "100vh",
        position: "fixed",
        top: 0,
        left: 0,
        zIndex: 1,
      }
    : {};

  return (
    <Box
      sx={{
        width: "fit-content",
        height: "fit-content",
        position: "relative",
        ...styles,
      }}
    >
      {isLoading && (
        <Box
          sx={{
            width: "100%",
            height: "100%",
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

- Trong `Loading` tạo `styled.ts`.

```jsx
// Loading/styled.ts
import { Box, styled } from "@mui/material";

export const LoadingOverlayStyled = styled(Box)({
  width: "100%",
  height: "100%",
  position: "absolute",
  top: 0,
  left: 0,
  backgroundColor: "white",
  zIndex: 1,
  display: "flex",
  alignItems: "center",
  justifyContent: "center",
});
```

- Trong folder `layouts`tạo `DetailPageLayout` dùng để bọc các detail page.
- Trong `DetailPageLayout/index.tsx`:
  - Nhận vào props `pageTitle`, `renderAction`, `renderContent`, `breadCrumbs`,`isLoading`.
  - Dùng component `Loading` bọc ngoài layout.
  - Dùng `renderContent` để render content trog detail page.
  - Dùng `renderAction` để render thêm các actions cần thiết.

```jsx
// DetailPageLayout/index.tsx
import Breadcrumb from "@/components/Breadcrumb";
import Loading from "@/components/Loading";
import { Box } from "@mui/material";
import React, { ReactNode } from "react";
import { BoxHeadStyled } from "./styled";

type DetailPageLayoutProps = {
  pageTitle: string,
  renderAction?: ReactNode,
  renderContent: ReactNode,
  isLoading?: boolean,
  breadCrumbs: { title: string, to?: string }[],
};

const DetailPageLayout: React.FC<DetailPageLayoutProps> = ({
  pageTitle,
  renderAction,
  renderContent,
  isLoading = false,
  breadCrumbs,
}) => {
  return (
    <Loading isLoading={isLoading} isLoadingForPage styles={{ width: "100%" }}>
      <Breadcrumb title={pageTitle} items={breadCrumbs || []} />
      <BoxHeadStyled>{renderAction}</BoxHeadStyled>
      <Box>{renderContent}</Box>
    </Loading>
  );
};

export default DetailPageLayout;
```

- Trong `DetailPageLayout` tạo file `styled.ts` để style cho layout.

```jsx
// DetailPageLayout/styled.ts
import { Box, styled } from "@mui/material";

export const BoxHeadStyled = styled(Box)({
  display: "flex",
  alignItems: "center",
  justifyContent: "space-between",
});

export const BoxLeftHeadStyled = styled(Box)({
  display: "inline-flex",
  alignItems: "center",
  gap: "8px",
});

export const ArrowStyled = styled(Box)({
  width: "24px",
  height: "24px",
  cursor: "pointer",
});
```
