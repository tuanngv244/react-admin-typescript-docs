# Setup course list page

- Dựng course detail page.
- Cây trúc thư mục.

```jsx
src
...
|_ pages
    |_ Course
        |_ components
            |_ContentSection.tsx
            |_GeneralInfoSection.tsx
            |_RequiredSection.tsx
            |_ScheduleSection.tsx
            |_UploadSection.tsx
            |_index.tsx
            |_styled.ts
        |_Detail
           |_index.tsx
           |_styled.ts
...
```

## Setup detail cơ bản page

- Trong `Course/Detail` tạo `index.tsx`.
- Trong `index.tsx`:
  - Dùng `courseService.getCourse` để get `courseDetailData`
  - Dùng `courseService.updateCourse` để update course.
  - Dùng `courseService.deleteCourseExecute` để delete course.
  - Tạo function `handleUpdateCourse`,`handleDeleteCourse`.
  - Tạo `breadCrumbs`.
  - Tạo `renderActions` dùng render các button actions.
  - Tạo `renderContent` dùng render content trong page.

```jsx
// Detail/index.tsx
import { Paths } from "@/constants/paths";
import { useMutation } from "@/hooks";
import DetailPageLayout from "@/layouts/DetailPageLayout";
import { courseService } from "@/services/course";
import { ICourse } from "@/types/course";
import { useEffect, useMemo, useRef } from "react";
import { useTranslation } from "react-i18next";
import { useNavigate, useParams } from "react-router-dom";

const CourseDetailPage = () => {
  const { t } = useTranslation();
  const { id } = useParams();
  const navigate = useNavigate();

  const {
    data: courseDetailData,
    execute: getCourseDetailExecute,
    loading: courseDetailLoading,
  } = useMutation<ICourse, string>(courseService.getCourse);
  const { execute: updateCourseExecute, loading: updateCourseLoading } =
    useMutation<ICourse, { id: string; formData: FormData }>(
      courseService.updateCourse
    );
  const { execute: deleteCourseExecute, loading: deleteCourseLoading } =
    useMutation<ICourse, string[]>(courseService.deleteCourse);

  const courseInitialValues: any = {};

  useEffect(() => {
    if (id) getCourseDetailExecute(id);
  }, [id]);

  const handleUpdateCourse = (data) => {
    // logic here...
  };
  const handleDeleteCourse = (id: string) => {
    // logic here...
  };

  const breadCrumbs = [
    {
      to: "/",
      title: "Home",
    },
    {
      to: Paths.COURSES,
      title: t("COURSE.listTitle"),
    },
    {
      title: t("COURSE.detailTitle"),
    },
  ];

  const renderActions = useMemo(
    () => <></>,
    [courseInitialValues, updateCourseLoading, deleteCourseLoading]
  );

  const renderContent = useMemo(
    () => <></>,
    [courseInitialValues, courseDetailLoading]
  );

  return (
    <DetailPageLayout
      pageTitle={""}
      renderContent={renderContent}
      breadCrumbs={breadCrumbs}
    />
  );
};

export default CourseDetailPage;

```

## Tạo xử lý các logic

- Dùng lệnh `yarn add @ckeditor/ckeditor5-react @ckeditor/ckeditor5-build-classic @mui/x-date-pickers` để cài đặt các package cần thiết.
- Dùng `handleUpdateCourse` thực thi update course.
- Dùng `handleDeleteCourse` thực thi delete course.
- Trong `renderActions` render button delete và save.
- Trong `renderContent` render `CourseForm`.

```jsx
import Dialog from "@/components/Dialog";
import { Paths } from "@/constants/paths";
import { useMutation } from "@/hooks";
import DetailPageLayout from "@/layouts/DetailPageLayout";
import { courseService } from "@/services/course";
import { ICourse, ICourseFormData } from "@/types/course";
import { Button } from "@mui/material";
import { useEffect, useMemo, useRef } from "react";
import { useTranslation } from "react-i18next";
import { useNavigate, useParams } from "react-router-dom";
import CourseForm, { TFormAction } from "../components/CourseForm";
import { BoxActionStyled } from "./styled";
import { mapCourseFormData } from "@/utils/pages/course";

const CourseDetailPage = () => {
  const formRef = useRef<TFormAction | null>(null);
  const { t } = useTranslation();
  const { id } = useParams();
  const navigate = useNavigate();

  const {
    data: courseDetailData,
    execute: getCourseDetailExecute,
    loading: courseDetailLoading,
  } = useMutation<ICourse, string>(courseService.getCourse);
  const { execute: updateCourseExecute, loading: updateCourseLoading } =
    useMutation<ICourse, { id: string; formData: FormData }>(
      courseService.updateCourse
    );
  const { execute: deleteCourseExecute, loading: deleteCourseLoading } =
    useMutation<ICourse, string[]>(courseService.deleteCourse);

  const courseInitialValues: any = {
    ...courseDetailData,
    teams: courseDetailData?.teams?.map((team) => team.id),
    content:
      courseDetailData?.content?.map((item) => ({
        title: item.title,
        description: item?.description?.join(","),
      })) || [],
  };

  useEffect(() => {
    if (id) getCourseDetailExecute(id);
  }, [id]);

  const handleUpdateCourse = (data: ICourseFormData) => {
    if (!courseDetailData?.id) return;
    const formData = mapCourseFormData(data);

    updateCourseExecute(
      { id: courseDetailData?.id, formData },
      {
        onSuccess: () => navigate(Paths.COURSES),
        onFailed: () => {},
      }
    );
  };
  const handleDeleteCourse = (id: string) => {
    deleteCourseExecute([id], {
      onSuccess: () => navigate(Paths.COURSES),
      onFailed: () => {},
    });
  };

  const breadCrumbs = [
    {
      to: "/",
      title: "Home",
    },
    {
      to: Paths.COURSES,
      title: t("COURSE.listTitle"),
    },
    {
      title: t("COURSE.detailTitle"),
    },
  ];

  const renderActions = useMemo(
    () => (
      <BoxActionStyled>
        <Dialog
          handleAgree={() =>
            handleDeleteCourse(courseInitialValues?.id as string)
          }
          renderButton={(toggle) => (
            <Button
              onClick={toggle}
              variant="outlined"
              color="error"
              fullWidth
              className="danger"
            >
              {t("COMMON.delete")}
            </Button>
          )}
          renderContent={
            <p>
              {t("COMMON.areYouSureWantToDelete", {
                name: courseInitialValues?.name,
              })}
            </p>
          }
        />

        <Button
          disabled={updateCourseLoading}
          onClick={() => formRef.current?.onSubmit()}
          type="submit"
          fullWidth
          variant="contained"
        >
          {t("COMMON.save")}
        </Button>
      </BoxActionStyled>
    ),
    [
      courseInitialValues,
      updateCourseLoading,
      deleteCourseLoading,
      formRef.current,
    ]
  );

  const renderContent = useMemo(
    () => (
      <CourseForm
        renderAction={renderActions}
        _onSubmit={handleUpdateCourse}
        initialDetailData={courseInitialValues}
        ref={formRef}
        isLoading={courseDetailLoading}
        isDetail
      />
    ),
    [courseInitialValues, courseDetailLoading]
  );

  return (
    <DetailPageLayout
      pageTitle={""}
      renderContent={renderContent}
      breadCrumbs={breadCrumbs}
    />
  );
};

export default CourseDetailPage;


```

- Trong folder `utils` tạo module `pages`, trong `pages` tạo file `course.ts`.
- Trong `pages/course.ts`:
  - Tạo function `mapCourseFormData` để map lại course theo server.

```jsx
// utils/pages/course.ts
import { ICourseFormData, ISchedule } from '@/types/course';

const mapCourseFormData = (data: ICourseFormData) => {
    const payload = {
        name: data.name,
        title: data.title,
        image: data.image,
        startDate: data.startDate,
        endDate: data.endDate,
        link: data.link,
        price: data.price,
        duration: data.duration,
        description: data.description,
        sortOrder: data.sortOrder,
        active: data.active,
    };

    if (typeof data.image === 'string') {
        delete payload.image;
    }

    const formData = new FormData();
    for (let key in data.schedule) {
        formData.append(`schedule[${key}]`, data.schedule[key as keyof ISchedule]);
    }
    for (let field in payload) {
        formData.append(field, payload[field as keyof typeof payload] as string | Blob);
    }
    for (let [index, value] of data.tags.entries()) {
        formData.append(`tags[${index}]`, value);
    }
    for (let [index, value] of data.required.entries()) {
        formData.append(`required[${index}]`, value);
    }
    for (let [index, value] of data.teams.entries()) {
        formData.append(`teams[${index}]`, value);
    }
    for (let [index] of data.content.entries()) {
        formData.append(`content[${index}][title]`, data.content[index].title);
        formData.append(`content[${index}][description]`, data.content[index].description);
    }

    return formData;
};

export { mapCourseFormData };

```

- Trong folder `store` Tạo module `instructor`.
- Trong `instructor` tạo file `index.ts`.
- Trong `instructor/index.ts`:
  - Khởi tạo `InstructorState`, `initialState`.
  - Khởi tạo `instructorReducer`.
  - `handleGetInstructors` để gọi api get instructors.
  - Trong `ex` handle các trường hợp `handleGetInstructors.fulfilled`,`handleGetInstructors.pending`,`handleGetInstructors.rejected`.
- Trong folder `services` ạo file `instructor.ts`:
- Trong `instructor.ts`:
  - Tạo `getInstructors`.
- Trong folder `types` tạo file `instructor.ts`.
- Trong `types/instructors.ts` định nghĩa type `IInstructor`,`IInstructorQuery`.

```jsx
import { instructorService } from "@/services/instructor";
import { AppState } from "@/store";
import { IInstructor } from "@/types/instructor";

import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";

type InstructorState = {
  instructors: IInstructor[] | [],
  loading: {
    getInstructors: boolean,
  },
};

const initialState: InstructorState = {
  instructors: [],
  loading: {
    getInstructors: false,
  },
};

const { actions, reducer: instructorReducer } = createSlice({
  name: "instructor",
  initialState,
  reducers: {
    reset() {
      return initialState;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(handleGetInstructors.fulfilled, (state, action) => {
        state.instructors = action.payload?.instructors;
        state.loading.getInstructors = false;
      })
      .addCase(handleGetInstructors.pending, (state) => {
        state.loading.getInstructors = true;
      })
      .addCase(handleGetInstructors.rejected, (state) => {
        state.loading.getInstructors = false;
      });
  },
});

export const handleGetInstructors = createAsyncThunk(
  "auth/getInstructors",
  async (_, thunkApi) => {
    try {
      const res = await instructorService.getInstructors({});
      return res?.data;
    } catch (error) {
      return thunkApi.rejectWithValue(error);
    }
  }
);

const instructorActions = { ...actions, handleGetInstructors };

const instructorsSelector = (state: AppState) => state.instructor.instructors;

export { instructorActions, instructorReducer, instructorsSelector };
```

```jsx
// services/instructor.ts
import { PaginationType } from '@/types/general';
import { IInstructor, IInstructorQuery } from '@/types/instructor';
import axiosInstance from '@/utils/axiosInstance';

const basePath = '/instructors' as const;

type TGetInstructorResponse = {
    instructors: IInstructor[];
    pagination: PaginationType;
};

export const instructorService = {
    getInstructors(payload: Partial<IInstructorQuery>): Promise<{ data: TGetInstructorResponse }> {
        return axiosInstance.get(`${basePath}`, { params: payload });
    },
};

```

```jsx
// types/instructor.ts
import { UseFormReturn } from 'react-hook-form';
import { IQuery } from './general';
import { CreateUpdateUser } from './user';

export interface IInstructorQuery extends IQuery {}

export interface IInstructor {
    id: string;
    name: string;
    slug: string;
    jobTitle: string;
    image: string;
    active: boolean;
    description: string;
    tags: string[];
    sortOrder: number;
    createdUser: CreateUpdateUser;
    updatedUser: CreateUpdateUser;
    createdAt: string;
    updatedAt: string;
}

export type TCreateUpdateInstructorUseFormReturn = UseFormReturn<any>;

export enum EInstructorTag {
    'MENTOR' = 'Mentor',
    'TEACHER' = 'Teacher',
}
```

- Trong folder `store` thêm `instructorReducer` vào `reducer` trong `configureStore`

```jsx
...

import { instructorReducer } from '@/store/modules/instructor';

export const store = configureStore({
    reducer: {
        instructor: instructorReducer,
    },
});

...

export default store;

```

- Trong `Course/components` tạo component `CourseForm`.
- Trong `CourseForm`:
  - Tạo `index.tsx`.
  - Tạo các component `ContentSection`,`GeneralInfoSection`,`RequiredSection`,`ScheduleSection` và `UploadSection`.
  - Tạo `styled.ts` dùng style cho các section và `CourseForm`.
- Trong `CourseForm/index.tsx`:
  - Tạo `initialValues`, `TFormAction` và `CourseFormProps`.
  - Tạo `isEdit` để quản lý trạng thái edit của form.
  - Dùng `form` để quản lý data của course detail.
  - Tạo `_onSubmit`.

```jsx
import Loading from "@/components/Loading";
import {
  BoxFormStyled,
  ListVerticalStyled,
  WrapDisableStyled,
} from "@/pages/Course/components/CourseForm/styled";
import { AppDispatch } from "@/store";
import { useTheme } from "@/theme";
import { ICourseFormData } from "@/types/course";
import { Box, Typography } from "@mui/material";
import {
  ReactNode,
  forwardRef,
  useEffect,
  useImperativeHandle,
  useState,
} from "react";
import { useForm } from "react-hook-form";
import { useDispatch } from "react-redux";
import ContentSection from "./ContentSection";
import GeneralInfoSection from "./GeneralInfoSection";
import RequiredSection from "./RequiredSection";
import ScheduleSection from "./ScheduleSection";
import UploadSection from "./UploadSection";
import { instructorActions } from "@/store/instructor/InstructorSlice";
import CustomSwitch from "@/components/CustomSwitch";

const initialValues: ICourseFormData = {
  name: "",
  title: "",
  startDate: undefined,
  endDate: undefined,
  duration: 0,
  tags: [],
  price: 0,
  link: "",
  schedule: undefined,
  teams: [],
  description: "",
  active: false,
  sortOrder: 1,
  content: [
    {
      title: "",
      description: "",
    },
  ],
  required: [""],
};

export type TFormAction = {
  reset: VoidFunction;
  onSubmit: VoidFunction;
  values: Partial<ICourseFormData>;
};

type CourseFormProps = {
  initialDetailData?: ICourseFormData;
  isLoading?: boolean;
  _onSubmit: (data: ICourseFormData) => void;
  renderAction: ReactNode;
  isDetail?: boolean;
};

const CourseForm = forwardRef(
  (
    {
      initialDetailData,
      _onSubmit,
      isLoading = false,
      renderAction,
      isDetail,
    }: CourseFormProps,
    ref
  ) => {
    const dispatch = useDispatch<AppDispatch>();
    const form = useForm<ICourseFormData>({
      defaultValues: initialValues,
    });

    const [isEdit, setIsEdit] = useState(false);

    const theme = useTheme();
    const borderColor = theme.palette.divider;

    const { getValues, reset, handleSubmit } = form;

    useImperativeHandle(
      ref,
      () => ({
        reset: reset,
        onSubmit: handleSubmit(_onSubmit),
        values: getValues,
      }),
      [initialDetailData]
    );

    useEffect(() => {
      dispatch(instructorActions.handleGetInstructors());
    }, []);

    useEffect(() => {
      if (initialDetailData) form.reset(initialDetailData);
    }, [initialDetailData]);

    const styleBoxSection = {
      padding: "0 15px 15px 15px",
      width: "100%",
      border: `1px solid ${borderColor}`,
    };

    return (
      <>
        <Box
          sx={{
            display: "flex",
            alignItems: "center",
            paddingTop: "20px",
            paddingLeft: "4px",
            marginLeft: "-12px",
          }}
        >
          {isDetail && (
            <CustomSwitch
              onChange={() => setIsEdit(!isEdit)}
              checked={isEdit}
            />
          )}
          <Typography variant="h4" sx={{ marginTop: "-4px" }}>
            {isEdit ? "Edit mode" : "View only mode"}
          </Typography>
        </Box>
        <Loading isLoading={isLoading}>
          <BoxFormStyled
            component={"form"}
            onSubmit={handleSubmit(_onSubmit)}
            sx={{ marginTop: "20px" }}
          >
            {isDetail && !isEdit && <WrapDisableStyled />}
            <ListVerticalStyled sx={{ width: "70%" }}>
              <Box sx={styleBoxSection}>
                <GeneralInfoSection form={form} />
              </Box>
              <Box sx={styleBoxSection}>
                <RequiredSection form={form} />
              </Box>
              <Box sx={styleBoxSection}>
                <ContentSection form={form} />
              </Box>
            </ListVerticalStyled>
            <Box
              sx={{ ...styleBoxSection, width: "30%", height: "fit-content" }}
            >
              <ScheduleSection form={form} />
              <UploadSection form={form} />
              {renderAction}
            </Box>
          </BoxFormStyled>
        </Loading>
      </>
    );
  }
);

export default CourseForm;


```

- Trong folder `components` tạo component `CustomSwitch`.
- Trong `CustomSwitch` tạo file `index.tsx`

```jsx
// components/CustomSwitch/index.tsx
import React from "react";
import { styled } from "@mui/material/styles";
import { Switch } from "@mui/material";

const CustomSwitch = styled((props: any) => <Switch {...props} />)(
  ({ theme }) => ({
    "&.MuiSwitch-root": {
      width: "68px",
      height: "49px",
    },
    "&  .MuiButtonBase-root": {
      top: "6px",
      left: "6px",
    },
    "&  .MuiButtonBase-root.Mui-checked .MuiSwitch-thumb": {
      backgroundColor: "primary.main",
    },
    "& .MuiSwitch-thumb": {
      width: "18px",
      height: "18px",
      borderRadius: "6px",
    },

    "& .MuiSwitch-track": {
      backgroundColor: theme.palette.grey[200],
      opacity: 1,
      borderRadius: "5px",
    },
    "& .MuiSwitch-switchBase": {
      "&.Mui-checked": {
        "& + .MuiSwitch-track": {
          backgroundColor: "primary",
          opacity: 0.18,
        },
      },
    },
  })
);

export default CustomSwitch;
```

- Trong `GeneralInfoSection.tsx`:

```jsx
// CourseForm/GeneralInfoSection.tsx
import CkEditor5 from "@/components/Ckeditor5";
import Input from "@/components/Input";
import Label from "@/components/Label";
import { DATE_FORMATS } from "@/constants/format";
import { courseTypeOptions } from "@/constants/pages/course";
import { instructorsSelector } from "@/store/instructor/InstructorSlice";
import { TCreateUpdateUseFormReturn } from "@/types/course";
import { transformToOptions } from "@/utils/transform";
import {
  Box,
  Divider,
  FormControl,
  Grid,
  MenuItem,
  Select,
  TextField,
} from "@mui/material";
import { DatePicker } from "@mui/x-date-pickers";
import dayjs, { Dayjs } from "dayjs";
import { Controller } from "react-hook-form";
import { useTranslation } from "react-i18next";
import { useSelector } from "react-redux";

type GeneralInfoSectionProps = {
  form: TCreateUpdateUseFormReturn,
};

const GeneralInfoSection: React.FC<GeneralInfoSectionProps> = ({ form }) => {
  const { t } = useTranslation();
  const {
    watch,
    control,
    setValue,
    register,
    formState: { errors },
  } = form;

  const instructors = useSelector(instructorsSelector);
  const instructorOptions = transformToOptions(instructors);

  return (
    <>
      <h3>{t("COURSE.generalInfo")}</h3>
      <Divider />
      <Input
        fullWidth
        label={t("COURSE.name")}
        errorMessage={errors?.name?.message}
        inputProps={{
          ...register("name", {
            required: t("MESSAGE_ERROR.pleaseEnterName"),
          }),
        }}
      />
      <Grid container spacing={2}>
        <Grid item xs={6}>
          <Controller
            name="tags"
            control={control}
            rules={{
              required: t("MESSAGE_ERROR.pleaseEnterTags"),
            }}
            defaultValue={[]}
            render={({ field }) => (
              <FormControl fullWidth>
                <Input
                  fullWidth
                  label={t("COURSE.types")}
                  errorMessage={errors?.tags?.message}
                  {...field}
                  renderInput={(props: any) => {
                    return (
                      <Select
                        defaultValue={[]}
                        fullWidth
                        multiple
                        sx={{ marginTop: "0 !important" }}
                        {...props}
                      >
                        {courseTypeOptions.map((option, index) => (
                          <MenuItem key={index} value={option.value}>
                            {option.label}
                          </MenuItem>
                        ))}
                      </Select>
                    );
                  }}
                />
              </FormControl>
            )}
          />
        </Grid>
        <Grid item xs={6}>
          <Input
            fullWidth
            label={t("COURSE.price")}
            errorMessage={errors?.price?.message}
            inputProps={{
              ...register("price", {
                required: t("MESSAGE_ERROR.pleaseEnterPrice"),
              }),
            }}
            type="number"
          />
        </Grid>
      </Grid>
      <Grid container spacing={2}>
        <Grid item xs={6}>
          <Input
            fullWidth
            label={t("COURSE.title")}
            errorMessage={errors?.title?.message}
            inputProps={{
              ...register("title", {
                required: t("MESSAGE_ERROR.pleaseEnterTitle"),
              }),
            }}
          />
        </Grid>
        <Grid item xs={6}>
          <Controller
            name="teams"
            control={control}
            rules={{
              required: t("MESSAGE_ERROR.pleaseEnterInstructor"),
            }}
            defaultValue={[]}
            render={({ field }) => (
              <FormControl fullWidth>
                <Input
                  label={t("COURSE.instructor")}
                  errorMessage={errors?.teams?.message}
                  {...field}
                  renderInput={(props: any) => {
                    return (
                      <Select
                        defaultValue={[]}
                        fullWidth
                        multiple
                        sx={{
                          marginTop: "0 !important",
                        }}
                        {...props}
                      >
                        {instructorOptions?.map((option, index) => (
                          <MenuItem key={index} value={option.value}>
                            {option.label}
                          </MenuItem>
                        ))}
                      </Select>
                    );
                  }}
                />
              </FormControl>
            )}
          />
        </Grid>
      </Grid>

      <Box>
        <Label> {t("COURSE.description")}</Label>
        <CkEditor5
          data={watch("description")}
          handleChange={(data: string) => setValue("description", data)}
        />
      </Box>

      <Grid container spacing={2}>
        <Grid item xs={6}>
          <Controller
            name={"startDate"}
            control={control}
            rules={{
              required: t("MESSAGE_ERROR.pleaseEnterStartDate"),
            }}
            render={({ field: { onChange, ...restField } }) => {
              return (
                <Input
                  fullWidth
                  label={t("COURSE.startDate")}
                  errorMessage={errors?.startDate?.message}
                  renderInput={(_) => (
                    <DatePicker
                      value={dayjs(restField?.value)}
                      sx={{ width: "100%" }}
                      format={DATE_FORMATS.DATE}
                      onChange={(value: Dayjs | null) =>
                        setValue(
                          "startDate",
                          dayjs(value).format(DATE_FORMATS.DATE)
                        )
                      }
                      slots={{
                        textField: (params) => <TextField {...params} />,
                      }}
                    />
                  )}
                />
              );
            }}
          />
        </Grid>
        <Grid item xs={6}>
          <Controller
            name={"endDate"}
            control={control}
            rules={{
              required: t("MESSAGE_ERROR.pleaseEnterEndDate"),
            }}
            render={({ field: { onChange, ...restField } }) => {
              return (
                <Input
                  fullWidth
                  label={t("COURSE.endDate")}
                  errorMessage={errors?.endDate?.message}
                  renderInput={(_) => (
                    <DatePicker
                      value={dayjs(restField?.value)}
                      sx={{ width: "100%" }}
                      format={DATE_FORMATS.DATE}
                      onChange={(value: Dayjs | null) =>
                        setValue(
                          "endDate",
                          dayjs(value).format(DATE_FORMATS.DATE)
                        )
                      }
                      slots={{
                        textField: (params) => <TextField {...params} />,
                      }}
                    />
                  )}
                />
              );
            }}
          />
        </Grid>
      </Grid>
      <Grid container spacing={2}>
        <Grid item xs={6}>
          <Input
            fullWidth
            label={t("COURSE.link")}
            errorMessage={errors?.link?.message}
            inputProps={{
              ...register("link", {
                required: t("MESSAGE_ERROR.pleaseEnterLink"),
              }),
            }}
          />
        </Grid>
        <Grid item xs={6}>
          <Input
            fullWidth
            label={t("COURSE.duration")}
            errorMessage={errors?.duration?.message}
            inputProps={{
              ...register("duration", {
                required: t("MESSAGE_ERROR.pleaseEnterDuration"),
              }),
            }}
            type="number"
          />
        </Grid>
      </Grid>
    </>
  );
};

export default GeneralInfoSection;
```

- Trong `utils/transform` Tạo function `transformToOptions`.

```jsx
// utils/transform.ts
import { Option } from "@/types/general";

...

const transformToOptions = <D>(
  data: D[],
  configs?: { labelKey: keyof D; valueKey: keyof D }
): Option[] | [] => {
  const { labelKey, valueKey } = configs || {};
  if (!!!data?.length) return [];
  return data.map(
    (item) =>
      ({
        label: item[(labelKey || "name") as keyof D],
        value: item[(valueKey || "id") as keyof D],
      } as Option)
  );
};

...

export { transformToOptions };

```

- Trong `App.tsx` bọc `LocalizationProvider` bên ngoài routing.

```jsx
...

import { LocalizationProvider } from '@mui/x-date-pickers';
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs';

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

- Trong folder `components` thêm component `Ckeditor5`.
- Trong `Ckeditor5` tạo file `index.tsx`.
- Trong `Ckeditor5` tạo file `uploadAdapter.ts`.
- Trong folder `services` tạo file `general.ts` để tạo api dùng chung.

```jsx
// Ckeditor5/index.tsx
import ClassicEditor from "@ckeditor/ckeditor5-build-classic";
import { CKEditor } from "@ckeditor/ckeditor5-react";
import { Box, FormHelperText, InputLabel } from "@mui/material";
import React from "react";
import { MyCustomUploadAdapterPlugin } from "./uploadAdapter";

const MIN_MAX_HEIGHT_CONTENT = 300;

type CkEditor5Props = {
  data?: string,
  handleChange: (data: string) => void,
  label?: string,
  errorMessage?: string,
};

const CkEditor5: React.FC<CkEditor5Props> = ({
  label,
  data,
  handleChange,
  errorMessage,
}) => {
  return (
    <Box
      sx={{
        ".ck-editor": {
          ".ck-content": {
            minHeight: MIN_MAX_HEIGHT_CONTENT,
            maxHeight: MIN_MAX_HEIGHT_CONTENT,
          },
        },
      }}
    >
      {label && (
        <InputLabel
          htmlFor={label}
          sx={{ position: "relative", marginBottom: "10px" }}
        >
          {label}
        </InputLabel>
      )}
      <CKEditor
        editor={ClassicEditor}
        data={data || ""}
        config={{
          extraPlugins: [MyCustomUploadAdapterPlugin],
        }}
        onReady={(_: unknown) => {}}
        onChange={(_: any, editor: ClassicEditor) => {
          const html = editor.getData();
          handleChange(html);
        }}
      />
      {errorMessage && (
        <FormHelperText
          error
          sx={{ fontSize: "14px" }}
          id="component-error-text"
        >
          {errorMessage}
        </FormHelperText>
      )}
    </Box>
  );
};

export default CkEditor5;
```

```jsx
// Ckeditor5/uploadAdapter.ts
import { BASE_URL } from "@/constants/environments";
import { generalService } from "@/services/general";
import tokenMethod from "@/utils/token";
import axios from "axios";

// function version ---> https://codesandbox.io/p/sandbox/react-ckeditor-upload-image-demo-cwcvh4?file=%2Fsrc%2FApp.tsx%3A40%2C2
function uploadAdapter(loader: FileLoader): UploadAdapter {
  return {
    upload: () => {
      return new Promise(async (resolve, reject) => {
        try {
          const file = await loader.file;
          const res = await generalService.uploadImage({ image: file });
          resolve({
            default: res?.data?.url,
          });
        } catch (error) {
          reject(error);
        }
      });
    },
    abort: () => {},
  };
}

function MyCustomUploadAdapterPlugin(editor: any) {
  editor.plugins.get("FileRepository").createUploadAdapter = (loader: any) =>
    uploadAdapter(loader);
}

export { MyCustomUploadAdapterPlugin };
```

```jsx
// services/general.ts
import { BASE_URL } from "@/constants/environments";
import axios from "axios";

export const generalService = {
  uploadImage(payload: File): Promise<{ data: { url: string } }> {
    return axios.post(
      `${BASE_URL.replace("/admin", "")}/ckeditor/upload/single`,
      payload,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );
  },
};
```

- Trong `RequiredSection.tsx`:
  - Tạo function `_onAddRequiredItem` ,`_onRemoveRequiredItem` để add/delete required item.

```jsx
// CourseForm/RequiredSection.tsx
import Input from "@/components/Input";
import { ListVerticalStyled } from "@/pages/Course/components/CourseForm/styled";
import { TCreateUpdateUseFormReturn } from "@/types/course";
import { Box, Divider, IconButton, Tooltip } from "@mui/material";
import { IconPlus, IconTrash } from "@tabler/icons-react";
import { useTranslation } from "react-i18next";

type RequiredSectionProps = {
  form: TCreateUpdateUseFormReturn,
};

const RequiredSection: React.FC<RequiredSectionProps> = ({ form }) => {
  const { t } = useTranslation();

  const { watch, register } = form;

  const _onAddRequiredItem = () => {
    form.setValue("required", [...(form.watch("required") || []), ""]);
  };
  const _onRemoveRequiredItem = (index: number) => {
    form.setValue(
      "required",
      (form.watch("required") || [])?.filter((_, idx) => idx !== index)
    );
  };

  return (
    <Box>
      <Box
        sx={{
          display: "flex",
          justifyContent: "space-between",
          alignItems: "center",
          width: "100%",
        }}
      >
        <h3>{t("COURSE.required")}</h3>
        <Box
          sx={{ cursor: "pointer" }}
          component={"div"}
          onClick={_onAddRequiredItem}
        >
          <IconPlus />
        </Box>
      </Box>
      <Divider sx={{ marginBottom: "20px" }} />
      <ListVerticalStyled>
        {watch("required")?.map((_, index) => (
          <Box
            width="100%"
            display="flex"
            alignItems="flex-center"
            gap="4px"
            key={index}
          >
            <Box width="100%">
              <Input
                inputProps={{ ...register(`required.${index}`) }}
                fullWidth
              />
            </Box>

            <Tooltip title="Delete" sx={{ width: "44px", height: "44px" }}>
              <IconButton
                aria-label="delete"
                size="small"
                onClick={() => _onRemoveRequiredItem(index)}
              >
                <IconTrash width={22} />
              </IconButton>
            </Tooltip>
          </Box>
        ))}
      </ListVerticalStyled>
    </Box>
  );
};

export default RequiredSection;
```

- Trong `ContentSection.tsx`:
  - Tạo function `_onAddContentItem` ,`_onRemoveContentItem` để add/delete content item.

```jsx
// CourseForm/ContentSection.tsx
import Input from "@/components/Input";
import { ListVerticalStyled } from "@/pages/Course/components/CourseForm/styled";
import { TCreateUpdateUseFormReturn } from "@/types/course";
import { Box, Divider, Grid, IconButton, Tooltip } from "@mui/material";
import { IconPlus, IconTrash } from "@tabler/icons-react";
import React from "react";
import { useTranslation } from "react-i18next";

type ContentSectionProps = {
  form: TCreateUpdateUseFormReturn,
};

const ContentSection: React.FC<ContentSectionProps> = ({ form }) => {
  const { t } = useTranslation();
  const { watch, register } = form;

  const _onAddContentItem = () => {
    form.setValue("content", [
      ...(form.watch("content") || []),
      {
        title: "",
        description: "",
      },
    ]);
  };
  const _onRemoveContentItem = (index: number) => {
    form.setValue(
      "content",
      (form.watch("content") || [])?.filter((_, idx) => idx !== index)
    );
  };

  return (
    <>
      <Box
        sx={{
          display: "flex",
          justifyContent: "space-between",
          alignItems: "center",
          width: "100%",
        }}
      >
        <h3>{t("COURSE.content")}</h3>
        <Box
          sx={{ cursor: "pointer" }}
          component={"div"}
          onClick={_onAddContentItem}
        >
          <IconPlus />
        </Box>
      </Box>
      <Divider sx={{ marginBottom: "-10px" }} />
      <ListVerticalStyled>
        {watch("content")?.map((_, index) => (
          <Grid container spacing={2} key={index}>
            <Grid item xs={5}>
              <Input
                fullWidth
                label={t("COURSE.title")}
                inputProps={{ ...register(`content.${index}.title`) }}
              />
            </Grid>
            <Grid item xs={7}>
              <Box width="100%" display="flex" alignItems="flex-end" gap="4px">
                <Box width="100%">
                  <Input
                    fullWidth
                    label={t("COURSE.description")}
                    inputProps={{ ...register(`content.${index}.description`) }}
                  />
                </Box>
                <Tooltip title="Delete" sx={{ width: "44px", height: "44px" }}>
                  <IconButton
                    aria-label="delete"
                    size="small"
                    onClick={() => _onRemoveContentItem(index)}
                  >
                    <IconTrash width={22} />
                  </IconButton>
                </Tooltip>
              </Box>
            </Grid>
          </Grid>
        ))}
      </ListVerticalStyled>
    </>
  );
};

export default ContentSection;
```

- Trong `ScheduleSection.tsx`

```jsx
// CourseForm/ScheduleSection.tsx
import Input from "@/components/Input";
import { TCreateUpdateUseFormReturn } from "@/types/course";
import { Box, Divider, Typography } from "@mui/material";
import { useTranslation } from "react-i18next";

type ScheduleSectionProps = {
  form: TCreateUpdateUseFormReturn,
};

const ScheduleSection: React.FC<ScheduleSectionProps> = ({ form }) => {
  const { t } = useTranslation();
  const {
    register,
    formState: { errors },
  } = form;
  return (
    <Box sx={{ marginBottom: "20px" }}>
      <h3>{t("COURSE.schedule")}</h3>
      <Divider />

      <Input
        fullWidth
        label={t("COURSE.startDate")}
        errorMessage={errors?.schedule?.startDate?.message}
        inputProps={{
          ...register("schedule.startDate", {
            required: t("MESSAGE_ERROR.pleaseEnterStartDate"),
          }),
        }}
      />
      <Input
        fullWidth
        label={t("COURSE.days")}
        errorMessage={errors?.schedule?.days?.message}
        inputProps={{
          ...register("schedule.days", {
            required: t("MESSAGE_ERROR.pleaseEnterDays"),
          }),
        }}
      />
      <Input
        fullWidth
        label={t("COURSE.time")}
        errorMessage={errors?.schedule?.time?.message}
        inputProps={{
          ...register("schedule.time", {
            required: t("MESSAGE_ERROR.pleaseEnterTime"),
          }),
        }}
      />

      <Input
        fullWidth
        label={t("COURSE.address")}
        errorMessage={errors?.schedule?.address?.message}
        inputProps={{
          ...register("schedule.address", {
            required: t("MESSAGE_ERROR.pleaseEnterAddress"),
          }),
        }}
      />
    </Box>
  );
};

export default ScheduleSection;
```

- Trong `UploadSection.tsx`
  - Dùng component `BoxUpload` để handle upload image.
- Trong folder `components` tạo component `BoxUpload`.
- Trong `BoxUpload` tạo file `index.ts` và `styled.ts`
- Trong `BoxUpload/index.ts`:
  - Tạo `_onFileChange` handle khi change file ảnh.
  - Tạo `_onDeleteImg` handle khi clear ảnh.
- Trong folder `utils` tạo file `validation.ts`.
- Trong `utils/validation.ts`:
  - Thêm function `validFileTypes` để validate định dạng file ảnh

```jsx
// CourseForm/UploadSection.tsx
import BoxUpload from '@/components/BoxUpload';
import { TCreateUpdateUseFormReturn } from '@/types/course';

type UploadSectionProps = {
    form: TCreateUpdateUseFormReturn;
};

const UploadSection: React.FC<UploadSectionProps> = ({ form }) => {
    return (
        <BoxUpload
            width={'100%'}
            height={50}
            initialImage={form.watch('image') as string}
            handleFileChange={(data: File) => form.setValue('image', data)}
            handleDeleteFile={() => form.setValue('image', undefined)}
        />
    );
};

export default UploadSection;
```

```jsx
// components/BoxUpload/index.tsx
import {
    BoxDeleteStyled,
    BoxPreviewImageStyled,
    BoxWrapStyled,
    InputUploadHidden,
} from '@/components/BoxUpload/styled';
import { validFileTypes } from '@/utils/validation';
import CloudUploadIcon from '@mui/icons-material/CloudUpload';
import DeleteIcon from '@mui/icons-material/Delete';
import { Button } from '@mui/material';
import React, { ChangeEvent, useEffect, useState } from 'react';

type BoxUploadProps = {
    handleFileChange: (data: File) => void;
    handleDeleteFile: VoidFunction;
    width?: number | string;
    height?: number;
    acceptTypes?: string[];
    initialImage?: string;
};

const BoxUpload: React.FC<BoxUploadProps> = ({
    handleFileChange,
    width = 200,
    height = 150,
    handleDeleteFile,
    initialImage,
}) => {
    const [imgPreview, setImgPreview] = useState<string | null>(initialImage || null);

    const _onFileChange = (ev: ChangeEvent<HTMLInputElement>) => {
        const file = ev.target?.files?.[0] as File;
        if (!validFileTypes(file)) return alert('Format file is invalid!');

        const reader = new FileReader();
        reader.readAsDataURL(file as File);
        reader.onloadend = function (_: unknown) {
            setImgPreview(reader.result as string);
        };
        handleFileChange(file);
    };

    const _onDeleteImg = () => {
        handleDeleteFile();
        setImgPreview(null);
    };

    useEffect(() => {
        (() => {
            if (initialImage) {
                setImgPreview(initialImage);
            }
        })();
    }, [JSON.stringify(initialImage)]);

    return (
        <BoxWrapStyled wbox={width} hbox={imgPreview ? 200 : height}>
            {imgPreview && (
                <React.Fragment>
                    <BoxPreviewImageStyled component={'img'} src={imgPreview as string} />
                    <BoxDeleteStyled className="btnDelete">
                        <Button onClick={_onDeleteImg} className="danger">
                            <DeleteIcon />
                        </Button>
                    </BoxDeleteStyled>
                </React.Fragment>
            )}
            {!imgPreview && (
                <React.Fragment>
                    <Button
                        component="label"
                        role={undefined}
                        variant="contained"
                        tabIndex={-1}
                        startIcon={<CloudUploadIcon />}
                        style={{ width: '100px', fontSize: '12px' }}
                    >
                        Upload
                        <InputUploadHidden
                            accept="image/*"
                            multiple
                            type="file"
                            onChange={_onFileChange}
                        />
                    </Button>
                </React.Fragment>
            )}
        </BoxWrapStyled>
    );
};

export default BoxUpload;
```

```jsx
// components/BoxUpload/styled.ts
import { Box, BoxProps, styled } from '@mui/material';

const BoxWrapStyled = styled(Box)<BoxProps & { wbox: number | string; hbox: number }>(
    ({ wbox, hbox }) => ({
        position: 'relative',
        width: wbox,
        height: hbox,
        borderRadius: '4px',
        overflow: 'hidden',
        border: `1px dashed #1D71F2`,
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        '&:hover': {
            '.btnDelete': {
                opacity: 1,
                visibility: 'visible',
                transition: 'all .25s',
            },
        },
    }),
);

const InputUploadHidden = styled('input')({
    clip: 'rect(0 0 0 0)',
    clipPath: 'inset(50%)',
    height: 1,
    overflow: 'hidden',
    position: 'absolute',
    bottom: 0,
    left: 0,
    whiteSpace: 'nowrap',
    width: 1,
});

const BoxPreviewImageStyled = styled(Box)<BoxProps & { src: string; component: string }>({
    width: '100%',
    height: '100%',
    objectFit: 'contain',
    position: 'absolute',
    top: 0,
    left: 0,
    borderRadius: '4px',
    overflow: 'hidden',
    padding: '0 4px',
});

const BoxDeleteStyled = styled(Box)({
    width: '100%',
    height: '100%',
    position: 'absolute',
    top: 0,
    left: 0,
    backgroundColor: 'rgba(0,0,0,0.3)',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    opacity: 0,
    visibility: 'hidden',
    transition: 'all .25s',
});

export { BoxWrapStyled, InputUploadHidden, BoxPreviewImageStyled, BoxDeleteStyled };

```

```jsx
// utils/validation.ts
const validFileTypes = (file: File, types?: string[]) => {
  const acceptTypes = types ?? ["png", "jpg", "jpeg"];
  const fileType = file?.type;
  const type = fileType?.replace("image/", "");
  return acceptTypes.includes(type);
};

export { validFileTypes };
```
