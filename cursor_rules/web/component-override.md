## Rhino UI Component Overrides: Authoring Guide for AI

This guide explains how to override Rhino UI components (inputs, headers, forms, displays, tables, etc.) using the project’s `app/frontend/rhino.config.jsx`. Use this when you need to customize rendering globally or for specific models.

Rhino UI customization docs: [General Configuration](https://www.rhino-project.org/docs/guides/ui/general), [Shell](https://www.rhino-project.org/docs/guides/ui/shell), [Sidebar](https://www.rhino-project.org/docs/guides/ui/sidebar), [Index page](https://www.rhino-project.org/docs/guides/ui/index_page), [Show page](https://www.rhino-project.org/docs/guides/ui/show_page), [Create page](https://www.rhino-project.org/docs/guides/ui/create_page), [Edit page](https://www.rhino-project.org/docs/guides/ui/edit_page).


### Where to configure overrides

- Root config file: `app/frontend/rhino.config.jsx`
- Compose overrides from:
  - `app/frontend/rhino-config/overrides/model.jsx`
  - `app/frontend/rhino-config/overrides/display.jsx`
  - `app/frontend/rhino-config/overrides/field.jsx`
  - `app/frontend/rhino-config/models.jsx` (per-model props and path-level overrides)

```jsx
// app/frontend/rhino.config.jsx
import { modelOverridesConfig } from './rhino-config/overrides/model';
import { displayOverridesConfig } from './rhino-config/overrides/display';
import { fieldOverridesConfig } from './rhino-config/overrides/field';
import { modelsConfig } from './rhino-config/models';
import { filterOverridesConfig } from './rhino-config/overrides/filter';

/** @type {import('@rhino-project/config').RhinoConfig} */
const rhinoConfig = {
  version: 1,
  components: {
    ...modelOverridesConfig,
    ...displayOverridesConfig,
    ...fieldOverridesConfig,
    ...filterOverridesConfig,
    ...modelsConfig
  }
};

export default rhinoConfig;
```


### Global overrides (apply to all models)

Use `model.jsx`, `display.jsx`, and `field.jsx` to override core building blocks across the entire app.

Example: replace model-level wrappers and lists globally:

```jsx
// app/frontend/rhino-config/overrides/model.jsx
export const modelOverridesConfig = {
  ModelCreate: MyCreateWrapper,
  ModelEdit: {
    ModelEditHeader: MyEditHeader,
    ModelEditForm: MyEditFormWrapper,
    ModelEditActions: MyEditActions
  },
  ModelShow: {
    ModelShowHeader: MyShowHeader,
    ModelShowDescription: MyShowDescriptionWrapper,
    ModelShowRelated: MyShowRelatedWrapper,
    ModelShowActions: null
  },
  ModelIndex: {
    ModelIndexTable: MyIndexTable,
    ModelIndexHeader: MyIndexHeader,
    ModelIndexActions: null
  },
  ModelFilters: MyModelFilters
};
```

- `ModelCreate`, `ModelEdit`, `ModelShow`, `ModelIndex`, and `ModelFilters` are top-level surfaces that compose internal components.
- Setting a sub-component (e.g., `ModelShowActions: null`) removes the default rendering.


### Overriding read-only displays

Read-only “displays” are used in show pages and read-only contexts. They are grouped by type. You can point a display group to any React component (it does not need to be an input; it can render plain text, a custom card, or any UI).

```jsx
// app/frontend/rhino-config/overrides/display.jsx
export const displayOverridesConfig = {
  DisplayLayoutVertical: {
    DisplayLabel: MyLabel
  },

  // Strings rendered via a custom component
  DisplayGroupString: {
    Display: MyStringDisplay
  },

  // Dates and booleans with opinionated renderers
  DisplayGroupDate: { Display: MyDateDisplay },
  DisplayGroupBoolean: { Display: MyBooleanDisplay }
};
```

Notes:
- Any display component receives the row/attribute context from Rhino (see docs). You are free to map props and compose them.
- You can override other groups: `Float`, `Integer`, `Text`, `DateTime`, `Time`, `SelectControlled`, `Reference`, `Enum`.


### Overriding form fields (inputs)

Field overrides redefine how editable inputs are rendered inside forms (create/edit). Similar to displays, fields are grouped by type.

```jsx
// app/frontend/rhino-config/overrides/field.jsx
export const fieldOverridesConfig = {
  FieldLayoutVertical: {
    FieldLabel: MyFieldLabel,
    FieldFeedback: MyFieldFeedback,
    FormGroup: MyFormGroup
  },

  FieldGroupString: { Field: MyStringInput },
  FieldGroupBoolean: { Field: MyBooleanInput },
  FieldGroupDate: { Field: MyDateInput },
  FieldGroupReference: { Field: MyReferenceInput }
};
```

Notes:
- Use `FieldLayoutVertical` to swap the label, feedback, and wrapper across all fields.
- Field groups include: `String`, `Float`, `Integer`, `Text`, `Date`, `DateTime`, `Boolean`, `Time`, `SelectControlled`, `Reference`, `Enum`, `File`.


### Per-model configuration

Use `models.jsx` to customize a specific model’s pages. You can:

- Restrict which fields appear on each page (Create, Edit, Show, Index, Filters) using the `paths` prop.
- Provide a custom component (e.g., `<ModelCell />` or a bespoke field/display) inline instead of a string path.

```jsx
// app/frontend/rhino-config/models.jsx
export const modelsConfig = {
  article: {
    ModelFilters: {
      props: { paths: ['created_at'] }
    },
    ModelCreate: {
      props: { paths: ['blog', 'title', 'status', 'image_attachment'] }
    },
    ModelEdit: {
      props: { paths: ['blog', 'title', 'status'] }
    },
    ModelShow: {
      props: { paths: ['blog', 'title', 'status', 'image_attachment'] }
    },
    ModelIndexTable: {
      props: { paths: ['title', 'status'] }
    }
  }
};
```

Inline component example (custom cell on index):

```jsx
import { ModelCell } from 'rhino/components/model_cell';

export const modelsConfig = {
  article: {
    ModelIndexTable: {
      props: {
        paths: [
          <ModelCell path="title" header="Summary" />,
          'image_attachment'
        ]
      }
    }
  }
};
```

Notes:
- The `paths` array order controls render order.
- On Show pages, you can similarly pass custom display components in place of string paths.


### Sidebar and Shell customization

- The Shell can be replaced with a custom component via `ApplicationShell`: [Shell](https://www.rhino-project.org/docs/guides/ui/shell)
- Sidebar models can be globally configured or role-organized via `PrimaryNavigation` props: [Sidebar](https://www.rhino-project.org/docs/guides/ui/sidebar)

Examples:

```js
const rhinoConfig = {
  version: 1,
  components: {
    ApplicationShell: MyCustomShell,
    PrimaryNavigation: { props: { models: ['blog', 'post'] } }
  }
};
```

Role-based sidebar:

```js
const rhinoConfig = {
  version: 1,
  components: {
    PrimaryNavigation: { props: { models: { admin: ['blog', 'post'], viewer: ['blog'] } } }
  }
};
```

Dynamic model list:

```js
const getModels = () => ['blog', 'post'];
const rhinoConfig = {
  version: 1,
  components: {
    PrimaryNavigation: { props: { models: getModels } }
  }
};
```


### Index and Show pages: column and display tweaks

- Index: Use `ModelIndexTable.props.paths` to choose columns and order; override headers/footers with `ModelHeader`/`ModelFooter` or cell-level `header`/`footer`: [Index page](https://www.rhino-project.org/docs/guides/ui/index_page)
- Show: Choose displayed fields via `ModelShow.props.paths` and switch to horizontal layout via `ModelDisplayGroup`: [Show page](https://www.rhino-project.org/docs/guides/ui/show_page)


### Create/Edit forms: field groups and actions

- Create: Configure with `ModelCreate.props.paths`; group or customize fields per model: [Create page](https://www.rhino-project.org/docs/guides/ui/create_page)
- Edit: Configure with `ModelEdit.props.paths` and override headers/actions/forms as needed: [Edit page](https://www.rhino-project.org/docs/guides/ui/edit_page)


### Best practices

- Prefer model-specific configs in `models.jsx` for per-resource tweaks; use global overrides for consistent cross-app patterns.
- Keep overrides small and composable; avoid duplicating complex UI across multiple areas—create shared components.
- Use inline components sparingly for readability; extract to reusable components when they grow.
- Document rationale in commit messages or local ADRs when replacing core Rhino components.


### Original Rhino components (reference for overrides)

When creating custom components that replace core Rhino UI, start from the original source and adapt. Copy the base component into your repository, fix imports/dependencies, and then edit as needed. This ensures compatibility with the expected props and composition.

- Model Edit surface:
  - ModelEdit: `https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelEdit.js`
  - ModelEditHeader: `https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelEditHeader.js`
  - ModelEditForm: `https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelEditForm.js`
  - ModelEditActions: `https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelEditActions.js`

Guidance:
- If a user asks to override `ModelEditForm`, show the corresponding base code above, copy it into the project, and adapt.
- Do the same for `ModelEditHeader`, `ModelEditActions`, or the entire `ModelEdit` surface if a larger rework is required.

For other surfaces (Show, Create, Index, etc.), browse the components tree and use the relevant files as your starting point:

- Components index: `https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components`

After copying, wire the new component via `app/frontend/rhino.config.jsx` (global) or `app/frontend/rhino-config/models.jsx` (per-model) as described earlier in this guide.


### Component catalog (what you can override)

Browse the upstream components to choose the exact surface to override: `https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models`

Common families and how to wire them via `rhino.config.jsx` keys:

- Create
  - Components: `ModelCreate`, `ModelCreateHeader`, `ModelCreateForm`, `ModelCreateActions`, `ModelCreateSimple`, `ModelCreateModal`, `ModelCreateModalActions`, `ModelCreateProvider`
  - Wiring keys: `ModelCreate`, `ModelCreateHeader`, `ModelCreateForm`, `ModelCreateActions`

- Edit
  - Components: `ModelEdit`, `ModelEditHeader`, `ModelEditForm`, `ModelEditActions`, `ModelEditSimple`, `ModelEditModal`, `ModelEditModalActions`, `ModelEditProvider`
  - Wiring keys: `ModelEdit`, `ModelEditHeader`, `ModelEditForm`, `ModelEditActions`

- Show
  - Components: `ModelShow`, `ModelShowHeader`, `ModelShowDescription`, `ModelShowRelated`, `ModelShowActions`, `ModelShowSimple`, `ModelShowProvider`
  - Wiring keys: `ModelShow`, `ModelShowHeader`, `ModelShowDescription`, `ModelShowRelated`, `ModelShowActions`

- Index/List
  - Components: `ModelIndex`, `ModelIndexHeader`, `ModelIndexTable`, `ModelIndexCardGrid`, `ModelIndexActions`, `ModelIndexSimple`, `ModelIndexProvider`, `ModelPager`, `ModelSearch`
  - Wiring keys: `ModelIndex`, `ModelIndexHeader`, `ModelIndexTable`, `ModelIndexActions`

- Filters
  - Components: `ModelFilters`, `ModelFilter`, `ModelFilterGroup`, `ModelFilterLabel`, `ModelFiltersSimple`, `ModelFiltersProvider`
  - Wiring keys: `ModelFilters`

- Fields (editable) and Displays (read-only)
  - Field components: `ModelFieldGroup`, `ModelFieldGroupFloating`, `ModelFieldGroupHorizontal`, `ModelFieldLabel`
  - Display components: `ModelDisplayGroup`, `ModelDisplayGroupFloating`, `ModelDisplayGroupHorizontal`, `ModelDisplayLabel`
  - Wiring keys: via `fieldOverridesConfig` (e.g., `FieldGroupString`, `FieldLayoutVertical`) and `displayOverridesConfig` (e.g., `DisplayGroupString`, `DisplayLayoutVertical`)

- Common building blocks
  - Components: `ModelHeader`, `ModelFooter`, `ModelSection`, `ModelCell`, `ModelProvider`
  - Wiring keys: `ModelHeader`, `ModelFooter` (where supported), per-surface sub-keys, and inline `ModelCell` via `paths` in `models.jsx`

When a developer asks to override a specific item (e.g., `ModelCreateForm`), fetch the corresponding upstream source, show that base code, copy it into the local codebase, fix imports, then wire it in `rhino.config.jsx` or `models.jsx` as shown earlier.


### Detailed catalogs by category

- Filters (upstream directory): [models/filters](https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models/filters)
  - `ModelFilterBoolean.js`
  - `ModelFilterDate.js`
  - `ModelFilterDateTime.js`
  - `ModelFilterEnum.js`
  - `ModelFilterFloat.js`
  - `ModelFilterInteger.js`
  - `ModelFilterIntegerSelect.js`
  - `ModelFilterOwnerReference.js`
  - `ModelFilterOwnerReferenceTypeahead.js`
  - `ModelFilterReference.js`
  - `ModelFilterReferenceTypeahead.js`
  - `ModelFilterString.js`
  - `ModelFilterTime.js`
  - `ModelFilterYear.js`

- Field Groups (inputs) (upstream directory): [models/fieldGroups](https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models/fieldGroups)
  - `ModelFieldGroupArrayInteger.js`
  - `ModelFieldGroupArrayString.js`
  - `ModelFieldGroupBoolean.js`
  - `ModelFieldGroupCountry.js`
  - `ModelFieldGroupCurrency.js`
  - `ModelFieldGroupDate.js`
  - `ModelFieldGroupDateTime.js`
  - `ModelFieldGroupEnum.js`
  - `ModelFieldGroupFile.js`
  - `ModelFieldGroupFloat.js`
  - `ModelFieldGroupInteger.js`
  - `ModelFieldGroupIntegerSelect.js`
  - `ModelFieldGroupJoinSimple.js`
  - `ModelFieldGroupNested.js`
  - `ModelFieldGroupOwnerReference.js`
  - `ModelFieldGroupPhone.js`
  - `ModelFieldGroupReference.js`
  - `ModelFieldGroupString.js`
  - `ModelFieldGroupText.js`
  - `ModelFieldGroupTime.js`
  - `ModelFieldGroupYear.js`

- Display Groups (read-only) (upstream directory): [models/displayGroups](https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models/displayGroups)
  - `ModelDisplayGroupArray.js`
  - `ModelDisplayGroupArrayReference.js`
  - `ModelDisplayGroupAttachment.js`
  - `ModelDisplayGroupAttachmentImage.js`
  - `ModelDisplayGroupAttachments.js`
  - `ModelDisplayGroupBoolean.js`
  - `ModelDisplayGroupCurrency.js`
  - `ModelDisplayGroupDate.js`
  - `ModelDisplayGroupDateTime.js`
  - `ModelDisplayGroupEnum.js`
  - `ModelDisplayGroupFloat.js`
  - `ModelDisplayGroupInteger.js`
  - `ModelDisplayGroupReference.js`
  - `ModelDisplayGroupString.js`
  - `ModelDisplayGroupText.js`
  - `ModelDisplayGroupTime.js`

- Index Cells (upstream directory): [models/cells](https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models/cells)
  - `ModelCellArray.js`
  - `ModelCellArrayReference.js`
  - `ModelCellAttachment.js`
  - `ModelCellAttachmentDownload.js`
  - `ModelCellAttachmentImage.js`
  - `ModelCellAttachments.js`
  - `ModelCellBoolean.js`
  - `ModelCellBooleanIcon.js`
  - `ModelCellCountry.js`
  - `ModelCellCurrency.js`
  - `ModelCellDate.js`
  - `ModelCellDateTime.js`
  - `ModelCellDateTimeDistance.js`
  - `ModelCellEnum.js`
  - `ModelCellFloat.js`
  - `ModelCellInteger.js`
  - `ModelCellLinkEmail.js`
  - `ModelCellLinkTelephone.js`
  - `ModelCellReference.js`
  - `ModelCellString.js`
  - `ModelCellTime.js`
  - `ModelEditableCellBoolean.js`
  - `ModelEditableCellReference.js`
  - `ModellCellIdentifier.js`

Override mapping:
- Filters: override via `ModelFilters` surface or a global `filterOverridesConfig` if present; per-model filter fields via `ModelFilters.props.paths` in `models.jsx`.
- Field Groups: override via `fieldOverridesConfig` keys (e.g., `FieldGroupString`, `FieldGroupReference`, `FieldLayoutVertical`).
- Display Groups: override via `displayOverridesConfig` keys (e.g., `DisplayGroupString`, `DisplayGroupDate`, `DisplayLayoutVertical`).
- Cells: provide custom cells inline in `ModelIndexTable.props.paths` (per-model) or swap table/cell components globally in `modelOverridesConfig`.

### Copying components and updating dependencies

Best practice when overriding: copy the original Rhino component file into your project, then customize it. This preserves expected props and internal composition. Always update imports and dependencies.

- Source index: `https://github.com/rhino-project/rhino-project/tree/beta/packages/core/src/components/models`
- Common dependencies you may need to install/use:
  - `@tanstack/react-table`
  - `lodash-es`
  - `prop-types`
  - Rhino package hooks/utils/components via `@rhino-project/core/...`
- Adjust imports: replace internal relative imports from Rhino core (e.g., `../../hooks/...`) with package exports used in your app (e.g., `@rhino-project/core/hooks`, `@rhino-project/core/components/models`).
- After copying, wire the component in `app/frontend/rhino.config.jsx` (global) or `app/frontend/rhino-config/models.jsx` (per-model).

Example: simple `ModelIndexHeader` override (generic)

```jsx
import {
  ModelFilters,
  ModelPager
} from '@rhino-project/core/components/models';
import { useModelIndexContext, useOverrides } from '@rhino-project/core/hooks';

const defaultComponents = {
  ModelFilters,
  ModelPager
};

export const ModelIndexHeader = ({ overrides, ...props }) => {
  const { ModelFilters, ModelPager } = useOverrides(defaultComponents, overrides);

  const { model } = useModelIndexContext();

  return (
    <div className={`model-index-header model-index-header-${model.model}`}>
      <div className="d-flex flex-column">
        <ModelFilters {...props} />
        <div className="d-flex flex-row">
          <div className="ms-auto">
            <ModelPager {...props} />
          </div>
        </div>
      </div>
    </div>
  );
};
```

Original reference: [ModelIndexHeader.js](https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelIndexHeader.js)

Example: `ModelIndexTable` override (generic skeleton)

```jsx
import {
  createColumnHelper,
  getCoreRowModel,
  useReactTable
} from '@tanstack/react-table';
import {
  cloneElement,
  isValidElement,
  useCallback,
  useEffect,
  useMemo,
  useState
} from 'react';
import { filter, isString } from 'lodash-es';

import {
  useModelIndexContext,
  useOverrides,
  useBaseOwnerNavigation,
  usePaths
} from '@rhino-project/core/hooks';
import { getModelShowPath } from '@rhino-project/core/utils';
import {
  Table,
  ModelCell,
  ModelFooter,
  ModelHeader,
  ModelSection
} from '@rhino-project/core/components/models';

const defaultComponents = {
  ModelHeader,
  ModelCell,
  ModelFooter,
  Table
};

const columnHelper = createColumnHelper();

export const ModelIndexTable = ({ overrides, paths, sortPaths, ...props }) => {
  const { ModelHeader, ModelCell, ModelFooter, Table } = useOverrides(
    defaultComponents,
    overrides
  );
  const { isInitialLoading, model, order, resources, results, setOrder } =
    useModelIndexContext();
  const baseOwnerNavigation = useBaseOwnerNavigation();
  const [sorting, setSorting] = useState([]);

  const pathsOrDefault = useMemo(
    () => paths || filter(model.properties, (a) => a.type !== 'identifier').map((a) => a.name),
    [paths, model]
  );
  const computedPaths = usePaths(pathsOrDefault, resources);

  const handleRowClick = useCallback(
    (row) => baseOwnerNavigation.push(getModelShowPath(model, row.original.id)),
    [baseOwnerNavigation, model]
  );

  const sortable = useMemo(
    () =>
      sortPaths ||
      filter(model.properties, (a) => ['string', 'datetime', 'float', 'integer'].includes(a.type)).map((a) => a.name),
    [sortPaths, model]
  );

  const columns = useMemo(
    () =>
      computedPaths.map((path, idx) => {
        if (isValidElement(path)) {
          const accessor = path.props?.accessor || (isString(path.props?.path) ? path.props?.path : null);
          const id = path.props?.id || path.props?.path || idx.toString();
          const header = path.props?.header || (() => (
            <ModelHeader model={model} path={path?.props?.path || null} />
          ));
          const cell = (cellProps) => cloneElement(path, { model, ...cellProps });
          const footer = path.props?.footer || (() => (
            <ModelFooter model={model} path={path?.props?.path || null} />
          ));

          return accessor
            ? columnHelper.accessor(accessor, {
                id,
                header,
                cell,
                footer,
                enableSorting: sortable.includes(id),
                enableMultiSort: sortable.includes(id)
              })
            : columnHelper.display({
                id,
                header,
                cell,
                footer,
                enableSorting: false
              });
        }

        const header = (info) => <ModelHeader model={model} path={path} {...info} />;
        const cell = (info) =>
          isInitialLoading ? (
            <div className="placeholder-glow">
              <span className="placeholder col-6"></span>
            </div>
          ) : (
            <ModelCell model={model} path={path} {...info} />
          );
        const footer = (info) => <ModelFooter model={model} path={path} {...info} />;

        return columnHelper.accessor(path, {
          id: path,
          header,
          cell,
          footer,
          enableSorting: sortable.includes(path),
          enableMultiSort: sortable.includes(path)
        });
      }),
    [computedPaths, isInitialLoading, model, sortable]
  );

  useEffect(() => {
    if (sorting.length === 0 && order) {
      setSorting(
        order.split(',').map((o) => ({ id: o.replace('-', ''), desc: o.startsWith('-') }))
      );
      return;
    }
    setOrder(sorting.map((o) => (o.desc ? '-' + o.id : o.id)).join(','));
  }, [order, setOrder, sorting]);

  const table = useReactTable({
    data: results || Array(10).fill({}),
    columns,
    getCoreRowModel: getCoreRowModel(),
    enableMultiSort: true,
    enableSortingRemoval: false,
    manualSorting: true,
    state: { sorting },
    onSortingChange: setSorting
  });

  return (
    <ModelSection baseClassName="index-table">
      <Table table={table} onRowClick={handleRowClick} {...props} />
    </ModelSection>
  );
};
```

Original reference: [ModelIndexTable.js](https://github.com/rhino-project/rhino-project/blob/beta/packages/core/src/components/models/ModelIndexTable.js)

Reminder: These examples are intentionally generic. In actual projects, copy the original Rhino component file, update imports and dependencies, then customize only the parts you need.


