# Setup common hooks

- Dựng các common hook như `useSearchQuery`, `useQuery`, `useMutation` và `useBoolean`.

## Setup useSearchQuery

- Dựng hook `useSearchQuery` để handle `searchParams` cho list page.
- Dùng lệnh `yarn add query-string` để cài đặt thư viện query-string.
- Trong folder `hooks` tạo hook `useSearchQuery.ts`.
- Trong `useSearchQuery.ts`:
  - Dùng `queryString.parse` để parse `queryObject`.
  - Khởi tạo `_onSetSearchParams` để set lại query params trên url.

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

## Setup useQuery

- Dựng hook `useQuery` để handle fetching data (thường được dùng cho các method: GET).
- Trong `useQuery`:
  - Nhận vào promise function để gọi api.
  - Nhận vào `options.preventInitialCall` để ngăn chặn gọi lần đầu khi dùng hook `useQuery`.

```jsx
// useQuery.ts
mport { useEffect, useState } from 'react';

export const useQuery = <
    D extends object | string | number | null = any,
    Q extends object | string | number = any,
>(
    promise: (...args: any) => Promise<{ data: D }>,
    dependencies?: unknown[],
    options?: {
        preventInitialCall: boolean;
    },
) => {
    const [data, setData] = useState<D>();
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<any>();

    useEffect(() => {
        if (options?.preventInitialCall) return;
        fetchData();
    }, [...(dependencies || []), options?.preventInitialCall]);

    const fetchData = async (query?: Q) => {
        setLoading(true);
        try {
            const res: { data: D } = await promise(query);
            setData(res?.data);
        } catch (error) {
            setError(error);
        } finally {
            setLoading(false);
        }
    };

    return {
        data,
        loading,
        error,
        refetch: fetchData,
    };
};
```

## Setup useMutation

- Dựng hook `useMutation` để handle fetching data (thường được dùng cho các method: POST,PUT, PATCH, DELETE ).
- Trong `useMutation`:
  - Nhận vào promise function để gọi api.
  - Trả về `execute` thực thi request.
  - Trong `execute` function, tham số thứ 2 truyền vào `options`.
  - Trong `options` chứa 2 method `onSuccess` và `onFailed` để bắt response của trạng thái thành công hay thất bại request.

```jsx
// useMutation.ts
import { useState } from 'react';

export const useMutation = <
    D extends object | string | number | null = any,
    Q extends object | string | number = any,
>(
    promise: (...args: any) => Promise<{ data: D }>,
) => {
    const [data, setData] = useState<D>();
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<any>();

    const execute = async (
        payload?: Q,
        options?: {
            onSuccess?: (data: D) => void;
            onFailed: (err: typeof error) => void;
        },
    ) => {
        const { onSuccess, onFailed } = options || {};
        try {
            setLoading(true);
            const res: { data: D } = await promise(payload);
            setData(res.data);
            onSuccess?.(res.data);
        } catch (error) {
            setError(error);
            onFailed?.(error);
        } finally {
            setLoading(false);
        }
    };

    return {
        execute,
        data,
        loading,
        error,
    };
};
```

## Setup useBoolean

- Dựng hook `useBoolean` dùng để kiểm soát trạng thái của 1 state boolean.
- Trong `useBoolean`:
  - Nhận vào `initialState` là boolean.
  - Các method `on`,`off`và `toggle` để để kiểm soát trạng thái value.

```jsx
// useBoolean.ts
import { useState } from "react";

export const useBoolean = (initialState?: boolean | (() => boolean)) => {
  const [value, setValue] = useState < boolean > (initialState ?? false);

  const on = () => setValue(true);
  const off = () => setValue(false);
  const toggle = () => setValue((prev) => !prev);
  const tools = {
    on: on,
    off: off,
    toggle: toggle,
  };
  const result: [typeof value, typeof tools] = [value, { on, off, toggle }];
  return result;
};
```

- Trong folder `hooks` tạo file `index.ts` để export các hooks.

```jsx
// hooks/index.ts
export * from "./useSearchQuery";
export * from "./useQuery";
export * from "./useMutation";
export * from "./useBoolean";
```
