# Setup page layout

- Dựng layout chung cho các page trong dự án bao như list page, detail page ,create/update page ,...
- Dựng common hook `useSearchQuery` để handle `searchParams` cho list page.

## Setup useSearchQuery

- Dùng lệnh `yarn add query-string` để cài đặt thư viện query-string.
- Trong folder `hooks` tạo hook `useSearchQuery.ts`.
- Trong `useSearchQuery.ts`:
  - Dùng `queryString.parse` để parse `queryObject`.
  - Khởi tạo `_onSetSearchParams` để set lại query params trên url.
- Trong folder `hooks` tạo file `index.ts` để export các hooks.

```jsx
// hooks/useSearchQuery.ts
import queryString from 'query-string';
import { useLocation } from 'react-router';
import { useSearchParams } from 'react-router-dom';

export const useSearchQuery = <P extends object = any>() => {
    const { search } = useLocation();
    const queryObject = queryString.parse(search) as Partial<P>;
    const [_, setSearchParams] = useSearchParams();

    const _onSetSearchParams = (query?: Partial<P>) => {
        const newQueryString = queryString.stringify({
            ...queryObject,
            ...query,
        });
        setSearchParams(new URLSearchParams(newQueryString));
    };

    return {
        searchParams: queryObject,
        _onSetSearchParams,
    };
};
```

```jsx
// hooks/index.ts
export * from "./useSearchQuery";
```

## Setup list page layout

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
import { Box, Button, Input } from "@mui/material";
import React, { ChangeEvent, ReactNode } from "react";
import { useTranslation } from "react-i18next";
import { useNavigate } from "react-router-dom";
import { BoxHeadStyled, BoxRightHeadStyled } from "./styled";
import _ from "lodash";
import { useSearchQuery } from "@/hooks";

interface ListPageLayoutProps {
  pageTitle: string;
  renderTable: ReactNode;
  createPath?: string;
  hideSearch?: boolean;
  searchPlaceholder?: string;
}

const ListPageLayout: React.FC<ListPageLayoutProps> = ({
  pageTitle,
  renderTable,
  createPath,
  hideSearch,
  searchPlaceholder,
}) => {
  const { t } = useTranslation();
  const navigate = useNavigate();
  const { searchParams, _onSetSearchParams } = useSearchQuery();
  const title = pageTitle.charAt(0).toUpperCase() + pageTitle.slice(1);
  const _onSearchChange = _.debounce((ev: ChangeEvent<HTMLInputElement>) => {
    _onSetSearchParams({ search: ev.target?.value });
  }, 500);

  return (
    <Box>
      <BoxHeadStyled>
        <h3>{title}</h3>
        <BoxRightHeadStyled>
          {!hideSearch && (
            <Input
              value={searchParams?.search}
              onChange={_onSearchChange}
              placeholder={searchPlaceholder}
            />
          )}
          <Button
            onClick={() =>
              navigate(createPath || location.pathname + "/create")
            }
          >
            {t("COMMON.create")} {pageTitle}
          </Button>
        </BoxRightHeadStyled>
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
import { Box, styled } from "@mui/material";

export const BoxHeadStyled = styled(Box)({
  display: "flex",
  alignItems: "center",
  justifyContent: "space-between",
});
export const BoxRightHeadStyled = styled(Box)({
  display: "inline-flex",
  alignItems: "center",
  gap: "8px",
});
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
  - Nhận vào props `pageTitle`, `renderAction`, `renderContent`, `backPath`,`isLoading`.
  - Dùng component `Loading` bọc ngoài layout.
  - Title uppercase kí tự đầu tiên.
  - Dùng `renderContent` để render content trog detail page.
  - Dùng `renderAction` để render thêm các actions cần thiết.

```jsx
// DetailPageLayout/index.tsx
import Loading from '@/components/Loading';
import ArrowBackIcon from '@mui/icons-material/ArrowBack';
import { Box } from '@mui/material';
import React, { ReactNode } from 'react';
import { To, useNavigate } from 'react-router-dom';
import { BoxHeadStyled, BoxLeftHeadStyled, ArrowStyled } from './styled';

type DetailPageLayoutProps = {
    pageTitle: string;
    renderAction?: ReactNode;
    renderContent: ReactNode;
    backPath?: string;
    isLoading?: boolean;
};

const DetailPageLayout: React.FC<DetailPageLayoutProps> = ({
    pageTitle,
    backPath,
    renderAction,
    renderContent,
    isLoading = false,
}) => {
    const navigate = useNavigate();
    const title = pageTitle.charAt(0).toUpperCase() + pageTitle.slice(1);

    return (
        <Loading isLoading={isLoading} isLoadingForPage styles={{ width: '100%' }}>
            <BoxHeadStyled>
                <BoxLeftHeadStyled>
                    <ArrowStyled onClick={() => navigate((backPath || -1) as To)}>
                        <ArrowBackIcon />
                    </ArrowStyled>
                    <h2>{title}</h2>
                </BoxLeftHeadStyled>
                {renderAction}
            </BoxHeadStyled>
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
