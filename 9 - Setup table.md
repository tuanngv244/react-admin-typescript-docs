# Setup table

- Dựng component `Table` dùng cho các list trong dự án.

## Setup table cơ bản với MUI

- Trong folder `components` tạo component `Table`
- Trong `Table`:
  - Dùng hàm `createData` để generate dummy data.
  - Dùng các components `Table`, `TableCell`, `TableBody`, `TableContainer`,... Để thiết kế component.

```jsx
// Table/index.tsx
import React from "react";
import Table from "@mui/material/Table";
import TableBody from "@mui/material/TableBody";
import TableCell from "@mui/material/TableCell";
import TableContainer from "@mui/material/TableContainer";
import TableHead from "@mui/material/TableHead";
import TableRow from "@mui/material/TableRow";
import Paper from "@mui/material/Paper";

function createData(
  name: string,
  calories: number,
  fat: number,
  carbs: number,
  protein: number
) {
  return { name, calories, fat, carbs, protein };
}

export type TableProps = {};
const Table: React.FC<TableProps> = ({}) => {
  const rows = [
    createData("Frozen yoghurt", 159, 6.0, 24, 4.0),
    createData("Ice cream sandwich", 237, 9.0, 37, 4.3),
  ];
  return (
    <TableContainer component={Paper}>
      <Table sx={{ minWidth: 650 }} aria-label="simple table">
        <TableHead>
          <TableRow>
            <TableCell>Dessert (100g serving)</TableCell>
            <TableCell align="right">Calories</TableCell>
            <TableCell align="right">Fat&nbsp;(g)</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {rows.map((row) => (
            <TableRow
              key={row.name}
              sx={{ "&:last-child td, &:last-child th": { border: 0 } }}
            >
              <TableCell component="th" scope="row">
                {row.name}
              </TableCell>
              <TableCell align="right">{row.calories}</TableCell>
              <TableCell align="right">{row.fat}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};

export default Table;
```

## Setup table nâng cao

- Dùng lệnh `yarn add dayjs @mui/x-date-pickers` để install thư viện `dayjs` và `@mui/x-date-pickers`.

- Trong `general.ts`:

  - Thêm các common types `PaginationType`,`ESortOrder`,`Option`.

- Trong folder `constants`:

  - Tạo file `format.ts` để format data.

- Nâng cấp component table với nhiều tính năng hơn như `search`, `sort`, `filter`, `select`, `render dynamic rows` và`render dynamic columns`.
- Trong component `Table`:
  - Thêm file `styled.ts` style cho table.
  - Định nghĩa type `ColumnsType` cho column có generic type `RecordType` (type record data).
  - Thêm các component hỗ trợ `TableRow`, `TableFilter`, `TableToolbar`.
  - Thêm các function `handleSelectedAll`, `handleSelectedChange`, `handleSortChange` để handle logic table.
- Trong folder `components` thêm component `Pagination` dùng thực hiện tính năng phân trang cho `Table`.
- Trong folder `types` thêm file `general.ts`.

```jsx
// types/general.ts
...

export type PaginationType = {
  total?: number,
  limit?: number,
  page?: number,
};

export type Option = {
    label: string;
    value: string;
};

export enum ESortOrder {
    ASC = '1',
    DESC = '-1',
}
...

```

- Trong `formats`.

```jsx
// constants/format.ts
export const DATE_FORMATS = {
  DATE: "MM-DD-YYYY",
};
```

```jsx
// Table/index.tsx
import {
    Box,
    Card,
    SxProps,
    TableBody,
    TableCell,
    Table as TableComponent,
    TableContainer,
    TableHead,
    TableRow as TableRowBase,
} from '@mui/material';
import { useTheme } from '@mui/material/styles';

import ArrowDownwardIcon from '@mui/icons-material/ArrowDownward';
import React, { ChangeEvent, ReactNode, useCallback, useMemo, useState } from 'react';

import { TFilterItem } from './TableFilter';
import TableRow from './TableRow';
import TableToolbar from './TableToolbar';
import Pagination from '@/components/Pagination';
import Loading from '@/components/Loading';
import Checkbox from '@/components/Checkbox';
import { WrapFilter } from '@/components/Table/styled';

export type ColumnsType<RecordType extends object = any> = {
    key: string;
    label?: string;
    sortKey?: string;
    columnStyles?: SxProps;
    headColumnStyles?: SxProps;
    disableClick?: boolean;
    columnWidth?: number; // Column width is calculated as a percentage base on the value passed. Ex: 4 ---> 4%
    render?(record: RecordType, index?: number): ReactNode;
    renderHead?(index?: number): ReactNode;
};

export type TableProps<RecordType extends object = any> = {
    columns: ColumnsType<RecordType>[];
    loading?: boolean;
    data: RecordType[] | null;
    wrapTableStyles?: SxProps;
    rowStyles?: SxProps;
    headStyles?: SxProps;
    hidePagination?: boolean;
    pagination?: PaginationType;
    hideSelecting?: boolean;
    handleDeleteSelected?: (selected: string[], onSuccess: VoidFunction) => void;
    filters?: TFilterItem[];
    titleTable?: string;
    hover?: boolean;
    onClickRows?: (record: RecordType) => void;
};

const Table: React.FC<TableProps> = ({
    columns,
    loading,
    data,
    wrapTableStyles,
    rowStyles,
    headStyles,
    hidePagination = false,
    pagination,
    hideSelecting,
    handleDeleteSelected,
    filters,
    titleTable,
    onClickRows,
    hover,
}) => {
    const theme = useTheme();
    const borderColor = theme.palette.divider;

    const handleSelectedAll = (ev: ChangeEvent<HTMLInputElement>) => {
        //Logic here
    };
    const handleSelectedChange = (recordId: string, checked: boolean) => {
        //Logic here
    };
    const handleSortChange = (sortKey: string) => {
        //Logic here
    };

    const isSelectedAll = false;

    const renderSortArrow = useCallback(
        (sortKey: string) => {
            return (
                <Box
                    onClick={() => handleSortChange(sortKey)}
                    className="sortArrow"
                    sx={{
                        position: 'absolute',
                        top: '0px',
                        right: '-30px',
                        opacity: 0,
                        visibility: 'hidden',
                        width: '20px',
                        height: '20px',
                        cursor: 'pointer',
                        transition: 'all .25s linear',
                        svg: {
                            width: 'inherit',
                            height: 'inherit',
                            transform: `rotate(${
                                searchParams?.orderBy === sortKey &&
                                searchParams?.order === ESortOrder.ASC
                                    ? -180
                                    : 0
                            }deg)`,
                            transition: 'all .25s linear',
                        },
                    }}
                >
                    <ArrowDownwardIcon />
                </Box>
            );
        },
        [searchParams],
    );

    return (
        <>
            {' '}
            <Card sx={{ padding: 0, border: `1px solid ${borderColor}` }}>
                <Box sx={wrapTableStyles}>
                    <TableToolbar
                        numSelected={selected?.length}
                        handleDeleteSelected={() =>
                            handleDeleteSelected?.(selected, () => setSelected([]))
                        }
                        filters={filters}
                        titleTable={titleTable}
                    />
                    <Loading isLoading={loading || false} styles={{ width: '100%' }}>
                        <TableContainer>
                            <TableComponent
                                sx={{
                                    maxHeight: 600,
                                    overflowY: data && data?.length > 10 ? 'scroll' : 'hidden',
                                    whiteSpace: 'nowrap',
                                }}
                                aria-label="table"
                            >
                                <TableHead
                                    sx={{
                                        ...headStyles,
                                    }}
                                >
                                    <TableRowBase>
                                        {!hideSelecting && (
                                            <TableCell>
                                                <Checkbox
                                                    checked={isSelectedAll}
                                                    onChange={handleSelectedAll}
                                                    tabIndex={-1}
                                                    inputProps={{
                                                        'aria-labelledby': 'select all desserts',
                                                    }}
                                                />
                                            </TableCell>
                                        )}
                                        {columns.map((column, idx) => {
                                            const {
                                                key,
                                                label,
                                                renderHead,
                                                headColumnStyles,
                                                sortKey,
                                            } = column;

                                            return typeof renderHead === 'function' ? (
                                                <TableCell key={key}>
                                                    {renderHead(idx)}
                                                    {sortKey && renderSortArrow(sortKey)}
                                                </TableCell>
                                            ) : (
                                                <TableCell
                                                    key={key}
                                                    sx={{
                                                        position: 'relative',
                                                        fontWeight: 600,
                                                        '&:hover': {
                                                            '.sortArrow': {
                                                                opacity: 1,
                                                                visibility: 'visible',
                                                            },
                                                        },
                                                        ...headColumnStyles,
                                                    }}
                                                >
                                                    <WrapFilter>
                                                        {label}
                                                        {sortKey && renderSortArrow(sortKey)}
                                                    </WrapFilter>
                                                </TableCell>
                                            );
                                        })}
                                    </TableRowBase>
                                </TableHead>
                                <TableBody>
                                    {data &&
                                        data?.map((record, idx) => {
                                            return (
                                                <TableRow
                                                    hover={hover}
                                                    key={idx}
                                                    data={record}
                                                    columns={columns}
                                                    rowStyles={rowStyles}
                                                    hideSelecting={hideSelecting}
                                                    isSelected={selected?.includes(record.id)}
                                                    handleSelectedChange={handleSelectedChange}
                                                    onClickRows={() => onClickRows?.(record)}
                                                />
                                            );
                                        })}
                                </TableBody>
                            </TableComponent>
                        </TableContainer>
                    </Loading>
                </Box>
            </Card>
            {!hidePagination && data && data.length > 0 && data?.length >= 10 && (
                <Pagination pagination={pagination} />
            )}
        </>
    );
};

export default Table;

```

- Trong file `Table/styled.ts`.

```jsx
// Table/styled.ts
import { Box, Popover, Toolbar, Typography, styled } from "@mui/material";

const BoxFilterIconStyled = styled(Box)({
  width: "20px",
  height: "20px",
  cursor: "pointer",
  svg: {
    width: "inherit",
    height: "inherit",
  },
});

const PanelPopoverStyled = styled(Popover)({
  ".MuiPopover-paper": {
    minWidth: "400px",
    borderRadius: "4px",
    boxShadow: "0px 12px 24px -4px #919EAB1F",
  },
});

const HeaderPanelStyled = styled(Box)({
  padding: "10px",
  borderBottom: "1px solid #e5eaef",
  borderRadius: "0",
  h6: {
    margin: 0,
    fontSize: "16px",
    fontWeight: 600,
    fontFamily: "Arial",
  },
});
const BodyPanelStyled = styled(Box)({
  padding: "10px",
  display: "flex",
  flexDirection: "column",
  gap: "12px",
});
const FooterPanelStyled = styled(Box)({
  display: "flex",
  alignItems: "center",
  justifyContent: "flex-end",
  borderTop: "1px solid #e5eaef",
  padding: "10px",
  gap: "4px",
  borderRadius: "0",
});

const LabelItemStyled = styled(Typography)({
  fontSize: "12px",
  marginBottom: "5px",
  fontFamily: "Arial",
});

const TypographyStyled = styled(Typography)({
  textTransform: "capitalize",
  fontWeight: "600",
});

const WrapToolbarStyled = styled(Toolbar)({
  minHeight: "56px !important",
  display: "flex",
  alignItems: "center",
  justifyContent: "space-between",
  borderBottom: "1px solid #e5eaef",
});
const WrapFilter = styled(`div`)({
  position: "relative",
  width: "fit-content",
  display: "flex",
});

export {
  BoxFilterIconStyled,
  PanelPopoverStyled,
  HeaderPanelStyled,
  BodyPanelStyled,
  FooterPanelStyled,
  LabelItemStyled,
  WrapToolbarStyled,
  TypographyStyled,
  WrapFilter,
};
```

- Trong `TableRow`:
  - Định nghĩa type props `TableRowProps`.
  - Dùng prop `hideSelecting` kiểm tra việc render checkbox cho row.
  - Dùng ` typeof render === 'function'` kiểm tra custom render từ bên ngoài và ngược lại render mặc định.
  - Thêm function `onClickRow` để gửi sự kiện click trên row
  - Thêm function `handleSelectedChange` khi có event checkbox.

```jsx
// Table/TableRow.tsx
import Checkbox from '@/components/Checkbox';
import { SxProps, TableCell, TableRow as TableRowBase } from '@mui/material';
import { ChangeEvent } from 'react';
import { ColumnsType } from '.';

type TableRowProps<RecordType extends object = any> = {
    data: RecordType;
    columns: ColumnsType<RecordType>[];
    rowStyles?: SxProps;
    hideSelecting?: boolean;
    isSelected?: boolean;
    hover?: boolean;
    handleSelectedChange?: (recordId: string, checked: boolean) => void;
    onClickRow?: () => void;
};

// enableLinkRows: clickable row in table

const TableRow: React.FC<TableRowProps> = ({
    data,
    columns,
    hideSelecting,
    rowStyles,
    handleSelectedChange,
    isSelected,
    hover,
    onClickRow,
}) => {
    return (
        <TableRowBase
            hover={hover}
            sx={{
                '&:last-child td, &:last-child th': { borderBottom: 0 },
                ...rowStyles,
            }}
        >
            {/* Using for case need render select record */}
            {!hideSelecting && (
                <TableCell>
                    <Checkbox
                        color="primary"
                        checked={isSelected}
                        onChange={(ev: ChangeEvent<HTMLInputElement>) =>
                            handleSelectedChange?.(data.id, ev.target?.checked)
                        }
                    />
                </TableCell>
            )}
            {columns.map((column, idx) => {
                const { render, columnStyles, columnWidth, disableClick } = column;

                const findData = data[column.key]; // auto required column.index to get info render data.

                // Using for case need custom render outside with record data
                if (render && typeof render === 'function')
                    return (
                        <TableCell
                            key={idx}
                            sx={{
                                width: `${columnWidth}%`,
                                wordBreak: 'break-word',
                                position: 'relative',
                                ...columnStyles,
                            }}
                            onClick={() => {
                                if (disableClick) {
                                    return;
                                }
                                onClickRow?.();
                            }}
                        >
                            {render(data)}
                        </TableCell>
                    );

                // Using for case need auto render follow get record data by index key
                return (
                    <TableCell
                        key={idx}
                        sx={{
                            width: `${columnWidth}%`,
                            wordBreak: 'break-word',
                            ...columnStyles,
                        }}
                        onClick={() => {
                            if (!disableClick) {
                                return;
                            }
                            onClickRow?.();
                        }}
                    >
                        {findData}
                    </TableCell>
                );
            })}
        </TableRowBase>
    );
};

export default TableRow;

```

- Trong `TableFilter`:
  - Định nghĩa type `TFilterItem`.
  - Nhận vào prop `filters` để render list filter được truyền khai báo từ bên ngoài và render theo 3 dạng `DATE`, `SELECT` và `MULTIPLE_SELECT`.
  - Thêm function `_onOpen`,`_onClose` kiểm soát việc bật/tắt panel filter.
  - Thêm function `_onSelectChange` khi có filter change.
  - Thêm function `_onReset` để clear filter data hiện tại.

```jsx
import {
    BodyPanelStyled,
    BoxFilterIconStyled,
    FooterPanelStyled,
    HeaderPanelStyled,
    LabelItemStyled,
    PanelPopoverStyled,
} from '@/components/Table/styled';
import { DATE_FORMATS } from '@/constants/format';
import { useSearchQuery } from '@/hooks';
import { Option } from '@/types/general';
import FilterAltIcon from '@mui/icons-material/FilterAlt';
import { Box, Button, MenuItem, Select, SelectChangeEvent, TextField } from '@mui/material';
import { DatePicker } from '@mui/x-date-pickers';
import dayjs, { Dayjs } from 'dayjs';
import React from 'react';
import { useTranslation } from 'react-i18next';

export type TFilterItem = {
    label: string;
    type: 'DATE' | 'SELECT' | 'MULTIPLE_SELECT';
    key: string;
    options?: Option[];
};

type TFilterProps = {
    filters: TFilterItem[];
};

const TableFilter: React.FC<TFilterProps> = ({ filters }) => {
    const { t } = useTranslation();
    const { searchParams, _onSetSearchParams } = useSearchQuery<{
        [key: string]: string;
    }>();
    const [anchorEl, setAnchorEl] = React.useState<HTMLDivElement | null>(null);
    const open = Boolean(anchorEl);

    const _onOpen = (event: React.MouseEvent<HTMLDivElement>) => {
        setAnchorEl(event.currentTarget);
    };

    const _onClose = () => {
        setAnchorEl(null);
    };

    const _onReset = () => {
        const filterKeys = filters.reduce((acc, curr) => {
            acc[curr.key] = undefined;
            return acc;
        }, {} as { [key: string]: undefined });

        _onSetSearchParams({ ...searchParams, ...filterKeys });
    };

    const _onSelectChange = (value: string, key: string) => {
        _onSetSearchParams({ ...searchParams, [key]: value });
    };

    return (
        <React.Fragment>
            <BoxFilterIconStyled component={'div'} onClick={_onOpen}>
                <FilterAltIcon />
            </BoxFilterIconStyled>
            <PanelPopoverStyled
                anchorEl={anchorEl}
                open={open}
                onClose={_onClose}
                anchorOrigin={{
                    vertical: 'bottom',
                    horizontal: 'left',
                }}
            >
                <HeaderPanelStyled>
                    <h6>{t('COMMON.filter')}</h6>
                </HeaderPanelStyled>
                <BodyPanelStyled>
                    {filters?.map((item, index) => {
                        const { type, options, key, label } = item;
                        if (type === 'SELECT')
                            return (
                                <Box key={index}>
                                    <LabelItemStyled>{label}</LabelItemStyled>
                                    <Select
                                        fullWidth
                                        value={searchParams[key] || ''}
                                        sx={{ marginTop: '0 !important' }}
                                        onChange={(ev: SelectChangeEvent) =>
                                            _onSelectChange(ev.target.value, key)
                                        }
                                    >
                                        {options?.map((option, index) => (
                                            <MenuItem key={index} value={option.value}>
                                                {option.label}
                                            </MenuItem>
                                        ))}
                                    </Select>
                                </Box>
                            );
                        if (type === 'DATE')
                            return (
                                <Box key={index}>
                                    <LabelItemStyled>{label}</LabelItemStyled>
                                    <DatePicker
                                        value={dayjs(searchParams?.[key])}
                                        sx={{ width: '100%' }}
                                        format={DATE_FORMATS.DATE}
                                        onChange={(value: Dayjs | null) =>
                                            _onSelectChange(
                                                dayjs(value).format(DATE_FORMATS.DATE),
                                                key,
                                            )
                                        }
                                        slots={{
                                            textField: (params) => <TextField {...params} />,
                                        }}
                                    />
                                </Box>
                            );
                        return <div key={index}></div>;
                    })}
                </BodyPanelStyled>
                <FooterPanelStyled>
                    <Button variant="outlined" color="error" onClick={_onClose}>
                        {t('COMMON.cancel')}
                    </Button>
                    <Button onClick={_onReset}>{t('COMMON.reset')}</Button>
                </FooterPanelStyled>
            </PanelPopoverStyled>
        </React.Fragment>
    );
};

export default TableFilter;

```

- Trong `TableToolbar`:

  - Định nghĩa type props `TableToolbarProps`.
  - Nhận vào prop `filters` và `numSelected`, dùng `filters` và `numSelected` kiểm tra render `TableFilter`.
  - Dùng `numSelected` kiểm tra render confirm `Dialog`.

- Trong folder `components` tạo component `Dialog`.

```jsx
import Dialog from "@/components/Dialog";
import { TypographyStyled, WrapToolbarStyled } from "@/components/Table/styled";
import { Box, Fab, Tooltip, Typography } from "@mui/material";
import { IconTrash } from "@tabler/icons-react";
import React from "react";
import { useTranslation } from "react-i18next";
import TableFilter, { TFilterItem } from "./TableFilter";

type TableToolbarProps = {
  numSelected: number,
  handleDeleteSelected: VoidFunction,
  filters?: TFilterItem[],
  titleTable?: string,
};

const TableToolbar: React.FC<TableToolbarProps> = ({
  numSelected,
  handleDeleteSelected,
  filters,
  titleTable,
}) => {
  const { t } = useTranslation();

  return (
    <WrapToolbarStyled>
      {numSelected > 0 ? (
        <Typography variant="h5">
          {numSelected} {t("COMMON.selected")}
        </Typography>
      ) : (
        <TypographyStyled variant="h5">
          {titleTable ?? t("COMMON.table")}
        </TypographyStyled>
      )}
      {numSelected > 0 && (
        <Dialog
          handleAgree={handleDeleteSelected}
          renderButton={(toggle) => (
            <Box
              sx={{
                cursor: "pointer",
              }}
              onClick={toggle}
            >
              <Tooltip title="Delete">
                <Fab color="error" aria-label="delete" size="small">
                  <IconTrash size={15} />
                </Fab>
              </Tooltip>
            </Box>
          )}
          renderContent={
            <Typography>
              {t("COMMON.areYouSureWantToDeleteSelected")}
            </Typography>
          }
        />
      )}
      {numSelected === 0 && filters && <TableFilter filters={filters} />}
    </WrapToolbarStyled>
  );
};

export default TableToolbar;
```

- Trong component `Dialog`:
  - Nhận vào các props `renderContent`,`renderButton`,`handleCancel`,`handleAgree`, để handle render và logic của Dialog.
  - Dùng các functions `on`, `off`, `toggle` để kiểm soát bật/tắt Dialog.

```jsx
// components/Dialog/index.tsx
import React, { ReactNode, forwardRef, useImperativeHandle } from "react";
import Button from "@mui/material/Button";
import DialogBase from "@mui/material/Dialog";
import DialogActions from "@mui/material/DialogActions";
import DialogContent from "@mui/material/DialogContent";
import DialogTitle from "@mui/material/DialogTitle";
import { useBoolean } from "@/hooks/useBoolean";
import { useTranslation } from "react-i18next";

type DialogProps = {
  title?: string,
  renderContent: ReactNode | (() => ReactNode),
  renderButton?: (toggle: VoidFunction) => ReactNode,
  handleCancel?: VoidFunction,
  handleAgree?: VoidFunction,
};

const Dialog = forwardRef((props: DialogProps, ref) => {
  const { t } = useTranslation();
  const { title, renderContent, handleAgree, handleCancel, renderButton } =
    props;
  const [open, { on, off, toggle }] = useBoolean();
  useImperativeHandle(
    ref,
    () => ({
      onDialog: on,
      offDialog: off,
      openDialog: open,
    }),
    []
  );

  return (
    <React.Fragment>
      {renderButton?.(toggle)}
      <DialogBase
        open={open}
        onClose={off}
        aria-labelledby="alert-dialog-title"
        aria-describedby="alert-dialog-description"
      >
        <DialogTitle id="alert-dialog-title">{title}</DialogTitle>
        <DialogContent>
          {typeof renderContent === "function"
            ? renderContent()
            : renderContent}
        </DialogContent>
        <DialogActions>
          <Button
            variant="outlined"
            color="error"
            onClick={() => {
              handleCancel?.();
              off();
            }}
          >
            {t("COMMON.cancel")}
          </Button>
          <Button
            variant="contained"
            onClick={() => {
              handleAgree?.();
              off();
            }}
            autoFocus
          >
            {t("COMMON.agree")}
          </Button>
        </DialogActions>
      </DialogBase>
    </React.Fragment>
  );
});

export default Dialog;
```

- Trong component `Pagination`:

  - Dùng `TablePagination` làm pagination cho dự án.
  - Thêm các function `handleChangeRowsPerPage`,`handleChangePage` để handle logic change page, change per page.
  - Dùng `_onSetSearchParams` để set lại url params khi change pagination data.

```jsx
// components/Pagination/index.tsx
import * as React from "react";
import TablePagination from "@mui/material/TablePagination";
import { PaginationType } from "@/types/general";
import { useSearchQuery } from "@/hooks/useSearchQuery";

type PaginationProps = {
  pagination?: PaginationType;
};

const Pagination = <T extends object = any>({
  pagination,
}: PaginationProps) => {
  const { searchParams, _onSetSearchParams } = useSearchQuery<
    { limit: number; page: number } & T
  >();

  const handleChangePage = (
    _: React.MouseEvent<HTMLButtonElement> | null,
    newPage: number
  ) => {
    const page = newPage + 1;
    // Default TablePagination receive page start is 0 so when we change paging need plus 1 more.
    _onSetSearchParams({ limit: searchParams?.limit, page } as T);
  };

  const handleChangeRowsPerPage = (
    event: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const limit = parseInt(event.target.value, 10);
    _onSetSearchParams({ limit: limit, page: searchParams?.page } as T);
  };

  return (
    <TablePagination
      component="div"
      count={pagination?.total || 0}
      page={Number(searchParams?.page || 1) - 1}
      onPageChange={handleChangePage}
      rowsPerPage={Number(searchParams?.limit || 10)}
      onRowsPerPageChange={handleChangeRowsPerPage}
    />
  );
};

export default Pagination;


```

## Thêm translation

```jsx
// languages/en.json
{
  ...,
      "COMMON": {
        "add": "Add",
        "cancel": "Cancel",
        "content": "Content",
        "create": "Create",
        "save": "Save",
        "edit": "Edit",
        "update": "Update",
        "delete": "Delete",
        "actions": "Actions",
        "action": "Action",
        "table": "Table",
        "agree": "Agree",
        "filter": "Filter",
        "reset": "Reset",
        "areYouSureWantToDelete": "Are you sure want to delete {{name}}?",
        "areYouSureWantToDeleteSelected": "Are you sure want to delete selected?",
        "selected": "Selected",
        "detail": "Detail"
    },
       "COURSE": {
        "listTitle": "course",
        "searchList": "Search for name course",
        "createTitle": "Create course",
        "generalInfo": "General info",
        "content": "Content",
        "required": "Required",
        "schedule": "Schedule",
        "detailTitle": "Detail course",
        "name": "Name",
        "types": "Types",
        "price": "Price",
        "teacher": "Teacher",
        "duration": "Duration",
        "description": "Description",
        "link": "Link",
        "startDate": "Start date",
        "openingDay": "Opening Day",
        "endDate": "End date",
        "title": "Title",
        "days": "Days",
        "time": "Time",
        "address": "Address",
        "instructor": "Instructor",
        "remove": "Remove",
        "course": "Course",
        "instructors": "Instructors",
        "dashboardIUpdateAfter5minutes!": "Dashboard information will be updated automatically every 5 minutes!"
    }
    ....
}
```

```jsx
// languages / vi.json;
{
  ...,
      "COMMON": {
        "add": "Add",
        "cancel": "Cancel",
        "content": "Content",
        "create": "Create",
        "save": "Save",
        "edit": "Edit",
        "update": "Update",
        "delete": "Delete",
        "actions": "Actions",
        "action": "Action",
        "table": "Table",
        "agree": "Agree",
        "filter": "Filter",
        "reset": "Reset",
        "areYouSureWantToDelete": "Are you sure want to delete {{name}}?",
        "areYouSureWantToDeleteSelected": "Are you sure want to delete selected?",
        "selected": "Selected",
        "detail": "Detail"
    },
       "COURSE": {
        "listTitle": "course",
        "searchList": "Search for name course",
        "createTitle": "Create course",
        "generalInfo": "General info",
        "content": "Content",
        "required": "Required",
        "schedule": "Schedule",
        "detailTitle": "Detail course",
        "name": "Name",
        "types": "Types",
        "price": "Price",
        "teacher": "Teacher",
        "duration": "Duration",
        "description": "Description",
        "link": "Link",
        "startDate": "Start date",
        "openingDay": "Opening Day",
        "endDate": "End date",
        "title": "Title",
        "days": "Days",
        "time": "Time",
        "address": "Address",
        "instructor": "Instructor",
        "remove": "Remove",
        "course": "Course",
        "instructors": "Instructors",
        "dashboardIUpdateAfter5minutes!": "Dashboard information will be updated automatically every 5 minutes!"
    }
    ....
}
```
