## Rhino Frontend API: Hooks and Custom Calls

This guide covers the primary Rhino frontend API mechanisms you will use most often. The core patterns are query hooks for reading and mutation hooks for writing, plus the ability to call custom endpoints for bespoke controllers.

Reference: [API Hooks](https://www.rhino-project.org/docs/reference/front_end/api_hooks)


### Query hooks

- `useModelShow(model, id, options)`
- `useModelIndex(model, options)`

Common options:
- `search`, `filter`, `order`, `limit`, `offset`
- `networkOptions` (axios options)
- `queryOptions` (react-query options such as `enabled`, `refetchInterval`)

Examples

```jsx
// Fetch details for a single resource
const { isSuccess, resource } = useModelShow('article', 23, {
  queryOptions: { enabled: true }
});
if (!isSuccess) return <Spinner/>;
```

```jsx
// Fetch a list with search, filter, ordering, and pagination
const { results, total } = useModelIndex('article', {
  search: 'Design',
  filter: { published: true, category: 'ux' },
  order: '-published_at,title',
  limit: 20,
  offset: 0
});
```

```jsx
// Disable query until a dependency is ready
const { results } = useModelIndex('article', {
  queryOptions: { enabled: !!user }
});
```

```jsx
// Refresh frequently
const { results } = useModelIndex('article', {
  queryOptions: { refetchInterval: 15_000 }
});
```


### Mutation hooks

- `useModelCreate(model, mutationOptions)`
- `useModelUpdate(model, mutationOptions)`
- `useModelDestroy(model, mutationOptions)`

Notes:
- Update and Destroy require an `id` in the payload passed to `mutate(...)`.
- The API returns the resulting object post-mutation.

Examples

```jsx
// Create
const { mutate: createArticle } = useModelCreate('article');
const onCreate = () => createArticle({ title: 'Hello' }, {
  onSuccess: (resp) => console.log('Created id', resp?.data?.id)
});
```

```jsx
// Update
const { mutate: updateArticle } = useModelUpdate('article');
const onSave = () => updateArticle({ id: 4, title: 'Updated' }, {
  onSuccess: (resp) => console.log('Updated title to', resp?.data?.title)
});
```

```jsx
// Destroy
const { mutate: deleteArticle } = useModelDestroy('article');
const onDelete = () => deleteArticle({ id: 4 }, {
  onSuccess: (resp) => console.log('Deleted', resp?.data?.id)
});
```


### Custom API calls (non-model endpoints)

When calling custom controllers (outside Rhino model routes), use your project’s network utility. Example pattern:

```jsx
useEffect(() => {
  const endpoint = `api/${organizationId}/dashboard/tasks`;
  networkApiCall(endpoint, { method: 'get' }).then((data) => {
    const formattedData = data.data.map((item) => ({
      status: formatStatus(item.status),
      color: statusColors[item.status],
      percentage: item.percentage,
      count: item.count
    }));
    setTaskStatuses(formattedData);
    setTimeout(() => setAnimate(true), 100);
  });
}, [organizationId]);
```

Tips:
- Ensure the backend route exists and is authorized per your policies.
- Use `networkOptions` or your network utility to add headers/tokens.


### Recipe: calendar view with multiple queries

```jsx
import { useMemo, useState } from 'react';
import { useBaseOwnerId, useModelIndex } from '@rhino-project/core/hooks';
import { startOfMonth, endOfMonth, format, subDays, addDays } from 'date-fns';
import FullCalendar from '@fullcalendar/react';
import dayGridPlugin from '@fullcalendar/daygrid';
import timeGridPlugin from '@fullcalendar/timegrid';
import interactionPlugin from '@fullcalendar/interaction';
import { useNavigate } from 'react-router-dom';

const TaskCalendar = () => {
  const baseOwnerId = useBaseOwnerId();
  const [currentDates, setCurrentDates] = useState({
    start: startOfMonth(new Date()),
    end: endOfMonth(new Date())
  });

  const { results: itemsWithDueDate } = useModelIndex('task', {
    filter: {
      due_date: {
        gteq: format(subDays(currentDates.start, 7), 'yyyy-MM-dd'),
        lteq: format(addDays(currentDates.end, 7), 'yyyy-MM-dd')
      },
      project: { organization: baseOwnerId }
    }
  });

  const { results: ongoingItems } = useModelIndex('task', {
    filter: {
      review_date: {
        gteq: format(subDays(currentDates.start, 7), 'yyyy-MM-dd'),
        lteq: format(addDays(currentDates.end, 7), 'yyyy-MM-dd')
      },
      project: { organization: baseOwnerId }
    }
  });

  const items = useMemo(
    () => [ ...(itemsWithDueDate || []), ...(ongoingItems || []) ],
    [itemsWithDueDate, ongoingItems]
  );

  const navigate = useNavigate();
  const events = useMemo(
    () =>
      items?.map((it) => ({
        id: it.id,
        title: it.title,
        start: it.due_date || it.review_date,
        allDay: true
      })) || [],
    [items]
  );

  return (
    <FullCalendar
      plugins={[dayGridPlugin, timeGridPlugin, interactionPlugin]}
      initialView="dayGridMonth"
      events={events}
      height="auto"
      headerToolbar={{ left: 'prev,next today', center: 'title', right: 'dayGridMonth,timeGridWeek' }}
      eventClick={(info) => navigate(`tasks/${info.event.id}`)}
      dateClick={() => navigate('tasks/new')}
      datesSet={(info) => setCurrentDates({ start: info.start, end: info.end })}
    />
  );
};

export default TaskCalendar;
```


### Recipe: profile update form with validation

```jsx
import { useCallback, useEffect, useMemo } from 'react';
import { Form } from 'reactstrap';
import * as yup from 'yup';
import { useForm } from 'react-hook-form';

import { FieldGroupString } from '@rhino-project/core/components/forms/fieldGroups';
import { FormProvider } from '@rhino-project/core/components/forms';
import {
  useFieldSetErrors,
  useModelShow,
  useResolver,
  useModelUpdate
} from '@rhino-project/core/hooks';
import { SubmitButton } from '@rhino-project/core/components/buttons';
import { DangerAlert, SuccessAlert } from '@rhino-project/core/components/alerts';

export const EditProfile = () => {
  const { model, resource: account } = useModelShow('account', null);
  const { mutate, isLoading, isSuccess, error } = useModelUpdate(model);

  const schema = useMemo(
    () =>
      yup.object().shape({
        name: yup.string().label('Name').ensure(),
        nickname: yup.string().label('Nickname').ensure()
      }),
    []
  );

  const defaultValues = useMemo(() => schema.default(), [schema]);
  const resolver = useResolver(schema);

  const methods = useForm({
    defaultValues,
    disabled: isLoading,
    values: account,
    mode: 'onBlur',
    resolver
  });
  const { handleSubmit, setError, setFocus, formState: { isDirty } } = methods;
  const onError = useFieldSetErrors(setError);
  const onSubmit = useCallback((values) => mutate(values, { onError }), [mutate, onError]);

  useEffect(() => setFocus('name'), [setFocus]);

  return (
    <>
      <FormProvider {...methods}>
        <Form onSubmit={handleSubmit(onSubmit)}>
          <FieldGroupString path="name" label="Name" />
          <FieldGroupString path="email" label="Email" />

          {Array.isArray(error?.errors) && <DangerAlert title={error.errors[0]} />}
          <SubmitButton loading={isLoading} disabled={!isDirty}>Update Profile</SubmitButton>
        </Form>
      </FormProvider>
      {isSuccess && <SuccessAlert title="Your profile has been updated successfully" />}
    </>
  );
};
```


### Best practices

- Prefer react-query status flags (`isSuccess`, `isLoading`, `isError`) over checking for `resource`/`results` nullability.
- Co-locate filters and ordering with the query; memoize derived data with `useMemo`.
- Use `queryOptions.enabled` for dependent queries.
- Use `networkOptions` to pass headers, params, or timeouts to axios.
- Build custom hooks from the primitives for repeated patterns (e.g., `usePublishedPosts`).

Reference: [API Hooks](https://www.rhino-project.org/docs/reference/front_end/api_hooks)


