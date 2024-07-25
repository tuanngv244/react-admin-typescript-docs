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
  return (
    <ListPageLayout
      searchPlaceholder={t("Search here")}
      pageTitle={t("COURSE.listTitle")}
      createPath={Paths.CREATE_COURSE}
      renderTable={<>render here...</>}
      breadCrumbs={breadCrumbs}
    />
  );
};

export default CourseListPage;
```

## Course table

```jsx
import Table, { ColumnsType } from '@/components/Table';
import { TFilterItem } from '@/components/Table/TableFilter';
import { DATE_FORMATS } from '@/constants/format';
import { Roles, courseTypeOptions } from '@/constants/pages/course';
import { Paths } from '@/constants/paths';
import { useMutation, useQuery } from '@/hooks';
import { useSearchQuery } from '@/hooks/useSearchQuery';
import { StyledChip, TypographyStyled } from '@/pages/Course/List/styled';
import { courseService } from '@/services/course';
import { ICourse, ICourseQuery } from '@/types/course';
import { PaginationType } from '@/types/general';
import { formatCurrency } from '@/utils/transform';
import { Avatar, Box, Fab, Stack, TableContainer, Tooltip } from '@mui/material';
import { IconEyeCog } from '@tabler/icons-react';
import dayjs from 'dayjs';
import React from 'react';
import { useTranslation } from 'react-i18next';
import { useNavigate } from 'react-router-dom';

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
        [JSON.stringify(searchParams)],
    );
    const { execute: deleteCourseExecute } = useMutation<ICourse, string[]>(
        courseService.deleteCourse,
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
            key: 'course',
            sortKey: 'course',
            label: t('COURSE.course'),
            columnWidth: 35,
            render: (record) => (
                <Stack direction="row" spacing={2} alignItems="center">
                    <Avatar
                        src={record.image}
                        alt={record.image}
                        sx={{
                            width: 80,
                            height: 50,
                            borderRadius: '6px',
                        }}
                    />
                    <TypographyStyled>{record.name}</TypographyStyled>
                </Stack>
            ),
        },
        {
            key: 'day',
            label: t('COURSE.openingDay'),
            columnWidth: 15,
            render: (record) => (
                <TypographyStyled>
                    {dayjs(record.startDate).format(DATE_FORMATS.DATE) || '---'}
                </TypographyStyled>
            ),
        },
        {
            key: 'tags',
            label: t('COURSE.types'),
            columnWidth: 15,
            render: (record) => (
                <TypographyStyled>
                    <StyledChip size="small" label={record.tags?.join(' | ') || '---'} />
                </TypographyStyled>
            ),
        },
        {
            key: 'price',
            sortKey: 'price',
            label: t('COURSE.price'),
            columnWidth: 25,
            render: (record) => (
                <TypographyStyled fontWeight={600} fontSize={'0.75rem'}>
                    {formatCurrency(record.price)}VND
                </TypographyStyled>
            ),
        },
        {
            key: 'type',
            label: t('COURSE.teacher'),
            columnWidth: 15,
            columnStyles: {
                textAlign: 'left',
            },
            render: (record) => {
                const teacher = record.teams?.find((team) => team.tags?.includes(Roles.Teacher));
                return <TypographyStyled>{teacher?.name || '---'}</TypographyStyled>;
            },
        },
        {
            key: 'duration',
            label: t('COURSE.duration'),
            columnWidth: 15,
            columnStyles: {
                textAlign: 'left',
            },
            render: (record) => {
                return <TypographyStyled textAlign={'center'}>{record.duration}</TypographyStyled>;
            },
        },
        {
            key: 'idx',
            label: '',
            columnWidth: 10,
            headColumnStyles: {
                textAlign: 'center',
            },

            render: (record) => (
                <Box
                    sx={{
                        display: 'flex',
                        alignItems: 'center',
                        justifyContent: 'center',
                        gap: '8px',
                    }}
                >
                    <Tooltip
                        placement="top"
                        title={t('COMMON.detail')}
                        onClick={() => navigate(Paths.COURSES + '/' + record.id)}
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
            label: t('COURSE.types'),
            key: 'types',
            options: courseTypeOptions,
            type: 'SELECT',
        },
    ];

    const onClickRows = (record: ICourse) => {
        navigate(Paths.COURSES + '/' + record.id);
    };

    return (
        <TableContainer>
            <Table
                columns={columns}
                data={courseData?.courses || []}
                loading={courseLoading}
                titleTable={''}
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
