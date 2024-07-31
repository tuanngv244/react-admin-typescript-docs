# Setup course list page

- Dựng course list page.
- Cây trúc thư mục.

```jsx
src
...
|_ pages
    |_ Course
        |_ components
        |_List
           |_index.tsx
           |_styled.ts
           |_CourseTable.tsx
        |_Detail
        |_Create
...
```

## Course list page

- Trong folder `pages` khởi tạo page `Course`.
- Trong `Course` khởi tạo `List`,`Detail`, `Create` and `components`.
- Trong `List` tạo `CourseListPage/index.ts`:

```jsx
import ListPageLayout from "@/layouts/ListPageLayout";
import { useTranslation } from "react-i18next";
import { Paths } from "@/constants/paths";
import CourseTable from "@/pages/Course/List/CourseTable";

const CourseListPage: React.FC = () => {
  const { t } = useTranslation();
  const renderTable = <CourseTable />;

  const breadCrumbs = [
    {
      to: "/",
      title: "Home",
    },
    {
      title: t("COURSE.listTitle"),
    },
  ];

  return (
    <ListPageLayout
      searchPlaceholder={t("Search here")}
      pageTitle={t("COURSE.listTitle")}
      createPath={Paths.CREATE_COURSE}
      renderTable={renderTable}
      breadCrumbs={breadCrumbs}
    />
  );
};

export default CourseListPage;
```

## Course table

- Trong folder `List` tạo component `CourseTable` để render course list.
- Trong `CourseTable`:
  - Dùng `useQuery` gọi request `courseService.getCourses` để lấy courseData.
  - Dùng `useMutation` gọi request `courseService.deleteCourse` để xóa course.
  - Khai báo `columns` để định nghĩa các column được render.
  - Khai báo `filters` làm lọc của course table.
  - Hàm `onClickRows` để redirect qua trang course detail.
  - Hàm `handleDeleteCourse` để xác nhận xóa course.
  - Dùng component `Table` để render courseData.
  - Dùng `TableContainer` của MUI để bọc component `Table`

```jsx
// List/CourseTable.tsx
import Table, { ColumnsType } from "@/components/Table";
import { TFilterItem } from "@/components/Table/TableFilter";
import { DATE_FORMATS } from "@/constants/format";
import { Paths } from "@/constants/paths";
import { useMutation, useQuery } from "@/hooks";
import { useSearchQuery } from "@/hooks/useSearchQuery";
import { courseService } from "@/services/course";
import { ICourse, ICourseQuery } from "@/types/course";
import { PaginationType } from "@/types/general";
import { formatCurrency } from "@/utils/transform";
import {
  Avatar,
  Box,
  Fab,
  Stack,
  TableContainer,
  Tooltip,
} from "@mui/material";
import { IconEyeCog } from "@tabler/icons-react";
import dayjs from "dayjs";
import React from "react";
import { useTranslation } from "react-i18next";
import { useNavigate } from "react-router-dom";
import { StyledChip, TypographyStyled } from "./styled";
import { courseTypeOptions, Roles } from "@/constants/pages/course";

const CourseTable: React.FC = () => {
  const { t } = useTranslation();
  const navigate = useNavigate();
  const { searchParams } = useSearchQuery<ICourseQuery>();

  const {
    data: courseData,
    loading: courseLoading,
    refetch,
  } = useQuery<{
    courses: ICourse[];
    pagination: PaginationType;
  }>(
    () => courseService.getCourses(searchParams || { page: 1, limit: 10 }),
    [JSON.stringify(searchParams)]
  );
  const { execute: deleteCourseExecute } = useMutation<ICourse, string[]>(
    courseService.deleteCourse
  );

  const handleDeleteCourse = (ids: string[]) => {
    deleteCourseExecute(ids, {
      onSuccess: refetch,
      onFailed: (error) => {
        //TODO:
      },
    });
  };

  const columns: ColumnsType<ICourse>[] = [
    {
      key: "course",
      sortKey: "course",
      label: t("COURSE.course"),
      columnWidth: 35,
      render: (record) => (
        <Stack direction="row" spacing={2} alignItems="center">
          <Avatar
            src={record.image}
            alt={record.image}
            sx={{
              width: 80,
              height: 50,
              borderRadius: "6px",
            }}
          />
          <TypographyStyled>{record.name}</TypographyStyled>
        </Stack>
      ),
    },
    {
      key: "day",
      label: t("COURSE.openingDay"),
      columnWidth: 15,
      render: (record) => (
        <TypographyStyled>
          {dayjs(record.startDate).format(DATE_FORMATS.DATE) || "---"}
        </TypographyStyled>
      ),
    },
    {
      key: "tags",
      label: t("COURSE.types"),
      columnWidth: 15,
      render: (record) => (
        <TypographyStyled>
          <StyledChip size="small" label={record.tags?.join(" | ") || "---"} />
        </TypographyStyled>
      ),
    },
    {
      key: "price",
      sortKey: "price",
      label: t("COURSE.price"),
      columnWidth: 25,
      render: (record) => (
        <TypographyStyled fontWeight={600} fontSize={"0.75rem"}>
          {formatCurrency(record.price)}VND
        </TypographyStyled>
      ),
    },
    {
      key: "type",
      label: t("COURSE.teacher"),
      columnWidth: 15,
      columnStyles: {
        textAlign: "left",
      },
      render: (record) => {
        const teacher = record.teams?.find((team) =>
          team.tags?.includes(Roles.Teacher)
        );
        return <TypographyStyled>{teacher?.name || "---"}</TypographyStyled>;
      },
    },
    {
      key: "duration",
      label: t("COURSE.duration"),
      columnWidth: 15,
      columnStyles: {
        textAlign: "left",
      },
      render: (record) => {
        return (
          <TypographyStyled textAlign={"center"}>
            {record.duration}
          </TypographyStyled>
        );
      },
    },
    {
      key: "idx",
      label: "",
      columnWidth: 10,
      headColumnStyles: {
        textAlign: "center",
      },

      render: (record) => (
        <Box
          sx={{
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            gap: "8px",
          }}
        >
          <Tooltip
            placement="top"
            title={t("COMMON.detail")}
            onClick={() => navigate(Paths.COURSES + "/" + record.id)}
          >
            <Fab size="small" color="primary" aria-label="small-bell">
              <IconEyeCog width={16} />
            </Fab>
          </Tooltip>
        </Box>
      ),
    },
  ];
  const filters: TFilterItem[] = [
    {
      label: t("COURSE.types"),
      key: "types",
      options: courseTypeOptions,
      type: "SELECT",
    },
  ];

  const onClickRows = (record: ICourse) => {
    navigate(Paths.COURSES + "/" + record.id);
  };

  return (
    <TableContainer>
      <Table
        columns={columns}
        data={courseData?.courses || []}
        loading={courseLoading}
        titleTable={""}
        pagination={{
          total: Number(courseData?.pagination?.total),
          limit: Number(searchParams?.limit),
          page: Number(searchParams?.page),
        }}
        handleDeleteSelected={handleDeleteCourse}
        filters={filters}
        hover={true}
        onClickRows={(record) => onClickRows?.(record)}
      />
    </TableContainer>
  );
};

export default CourseTable;


```

- Trong `List` tạo file `styled.ts`

```jsx
// List/styled.ts
import { Chip, ChipProps, Typography, styled } from "@mui/material";

const StyledChip =
  styled(Chip) <
  ChipProps >
  (({ theme }) => ({
    borderRadius: "8px",
    color: theme.palette.success.main,
    fontWeight: 100,
  }));

const TypographyStyled = styled(Typography)({
  fontSize: "0.75rem",
});

export { StyledChip, TypographyStyled };
```

- Trong `utils/transform.ts`:
  - Thêm function ``

```jsx
// utils/transform.ts
...

const formatCurrency = (data: number, configs?: { type: string }) => {
  return new Intl.NumberFormat("vi-VN", {
    currency: configs?.type || "VND",
  }).format(data || 0);
};

export {  formatCurrency };

...
```

- Trong folder `types`:
  - Tạo file type `general.ts` định nghĩa type dùng chung.
  - Tạo file type `course.ts` định nghĩa type cho course.
- Trong `types/general.ts`:
  - Định nghĩa type `IQuery`.
- Trong `types/course.ts`:
  - Định nghĩa type `ICourse`,`ISchedule`, `IContent` course item.
  - Định nghĩa type `ICourseQuery` cho query.
  - Định nghĩa enum `ECourseTypes`.

```jsx
//types/general.ts
export interface IQuery {
  page: number;
  limit: number;
  order: string;
  orderBy: string;
  dateFrom: string;
  dateTo: string;
  search: string;
  active: boolean;
}
```

```jsx
// types/course.ts
import { UseFormReturn } from "react-hook-form";
import { IQuery } from "./general";
import { CreateUpdateUser } from "./user";

export interface ICourseFormData {
  name: string;
  title: string;
  startDate?: string;
  endDate?: string;
  duration: number;
  tags: string[];
  price: number;
  link: string;
  schedule?: ISchedule;
  content: Array<{ title: string; description: string }>;
  required: string[];
  teams: string[];
  description: string;
  active: boolean;
  sortOrder: number;
  image?: string | File;
}

export interface ICourseQuery extends IQuery {}

export interface ICourse {
  id: string;
  name: string;
  slug: string;
  title: string;
  description: string;
  image: string;
  active: boolean;
  startDate: string;
  endDate: string;
  duration: number;
  tags: string[];
  price: number;
  link: string;
  schedule: ISchedule;
  content: IContent[];
  required: string[];
  teams: Array<{
    tags: string[];
    active: boolean;
    name: string;
    slug: string;
    image: string;
    jobTitle: string;
    description: string;
    sortOrder: number;
    createdUser: string;
    updatedUser: string;
    createdAt: string;
    updatedAt: string;
    id: string;
  }>;
  sortOrder: number;
  createdUser: CreateUpdateUser;
  updatedUser: CreateUpdateUser;
  createdAt: string;
  updatedAt: string;
}

export interface ISchedule {
  startDate: string;
  days: string;
  time: string;
  address: string;
}

export interface IContent {
  description: string[];
  title: string;
}

export enum ECourseTypes {
  "ONLINE" = "Online",
  "OFFLINE" = "Offline",
}

export type TCreateUpdateUseFormReturn = UseFormReturn<
  ICourseFormData,
  any,
  ICourseFormData
>;


...
```

- Trong folder `services` tạo file `course.ts`.
- Trong `services/course.ts`:
  - Export `courseService` để dùng api của course.
  - Tạo method `getCourses`.
  - Tạo method `deleteCourse`.

```jsx
// services/course.ts
import { ICourse, ICourseQuery } from "@/types/course";
import { PaginationType } from "@/types/general";
import axiosInstance from "@/utils/axiosInstance";

const basePath = "/courses" as const;

type TGetCourseResponse = { courses: ICourse[]; pagination: PaginationType };

export const courseService = {
  getCourses(
    payload: Partial<ICourseQuery>
  ): Promise<{ data: TGetCourseResponse }> {
    return axiosInstance.get(`${basePath}`, { params: payload });
  },
    deleteCourse(ids: string[]): Promise<{ data: ICourse }> {
        return axiosInstance.delete(`${basePath}`, { data: ids });
    },
};

```

- Trong folder `constants` tạo folder `pages`.
- Trong `pages` tạo file `course.ts`.
- Trong `pages/courses.ts`:
  - Thêm `courseTypeOptions`.
  - Thêm `Roles` .

```jsx
// constants/pages/course.ts
import { ECourseTypes } from "@/types/course";

const courseTypeOptions = [
  { label: "Online", value: ECourseTypes.ONLINE },
  { label: "Offline", value: ECourseTypes.OFFLINE },
];

const Roles = {
  Teacher: "Teacher",
  Mentor: "Mentor",
};

export { courseTypeOptions, Roles };
```

```jsx
// locales/en.json
{

...

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

...

}

```

```jsx
// locales/en.json
{

...
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

...

}

```

```jsx
// locales/vi.json

{

...

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

 ...

}

```
